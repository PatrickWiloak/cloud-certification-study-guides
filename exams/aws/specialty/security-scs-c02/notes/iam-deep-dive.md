# Identity and Access Management Deep Dive

## IAM Fundamentals

### IAM Components

**Users**
- Individual identities
- Long-term credentials
- Access keys for programmatic access
- Password for console access
- MFA for enhanced security

**Groups**
- Collections of users
- Simplify permission management
- Users inherit group permissions
- Users can belong to multiple groups

**Roles**
- Temporary credentials
- AssumeRole for identity federation
- Cross-account access
- Service roles for AWS services

**Policies**
- JSON documents defining permissions
- Attached to users, groups, roles
- Identity-based or resource-based
- Managed or inline

### IAM Policy Structure

**Policy Syntax**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "UniqueStatementId",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:user/Alice"
      },
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::example-bucket/*",
      "Condition": {
        "StringEquals": {
          "aws:username": "Alice"
        },
        "IpAddress": {
          "aws:SourceIp": "203.0.113.0/24"
        }
      }
    }
  ]
}
```

**Policy Elements**
- **Version**: Policy language version (always "2012-10-17")
- **Statement**: Array of permission statements
- **Sid**: Statement ID (optional, for documentation)
- **Effect**: Allow or Deny
- **Principal**: Who the policy applies to (resource-based only)
- **Action**: API operations allowed/denied
- **Resource**: AWS resources affected
- **Condition**: Optional constraints

### Policy Evaluation Logic

**Evaluation Flow**
```
1. Explicit DENY → Immediately deny
2. Organizations SCP → Must allow
3. Resource-based policy → Can allow
4. Permission boundaries → Sets maximum permissions
5. Session policies → Further restricts
6. Identity-based policy → Must allow
7. Implicit DENY → Default deny if no explicit allow
```

**Example Evaluation**
```json
// Identity-based policy
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "s3:*",
    "Resource": "*"
  }]
}

// Permission boundary (limits maximum permissions)
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": ["s3:GetObject", "s3:ListBucket"],
    "Resource": "*"
  }]
}

// Effective permissions = Intersection
// User can only GetObject and ListBucket, not PutObject
```

## Advanced IAM Patterns

### Permission Boundaries

**Use Cases**
- Delegate user creation to developers
- Prevent privilege escalation
- Enforce maximum permissions
- Multi-tenant environments

**Implementation**
```json
// Permission boundary policy
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ServiceBoundaries",
      "Effect": "Allow",
      "Action": [
        "s3:*",
        "dynamodb:*",
        "lambda:*"
      ],
      "Resource": "*"
    },
    {
      "Sid": "RegionRestriction",
      "Effect": "Allow",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": ["us-east-1", "us-west-2"]
        }
      }
    }
  ]
}
```

**Attach Boundary**
```bash
# Create user with permission boundary
aws iam create-user --user-name developer1

aws iam attach-user-policy \
  --user-name developer1 \
  --policy-arn arn:aws:iam::123456789012:policy/DeveloperAccess

aws iam put-user-permissions-boundary \
  --user-name developer1 \
  --permissions-boundary arn:aws:iam::123456789012:policy/DeveloperBoundary
```

### Service Control Policies (SCPs)

**Organization-Wide Restrictions**
```json
// Deny all actions in non-approved regions
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyUnapprovedRegions",
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": [
            "us-east-1",
            "us-west-2",
            "eu-west-1"
          ]
        },
        "ArnNotLike": {
          "aws:PrincipalArn": [
            "arn:aws:iam::*:role/OrganizationAccountAccessRole"
          ]
        }
      }
    }
  ]
}

// Deny instance types
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "DenyExpensiveInstances",
    "Effect": "Deny",
    "Action": "ec2:RunInstances",
    "Resource": "arn:aws:ec2:*:*:instance/*",
    "Condition": {
      "StringNotEquals": {
        "ec2:InstanceType": [
          "t2.micro",
          "t2.small",
          "t3.micro",
          "t3.small"
        ]
      }
    }
  }]
}

// Prevent leaving organization
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Deny",
    "Action": "organizations:LeaveOrganization",
    "Resource": "*"
  }]
}
```

### Attribute-Based Access Control (ABAC)

**Tag-Based Permissions**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ABACReadWrite",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::${aws:PrincipalTag/Department}/*",
      "Condition": {
        "StringEquals": {
          "s3:ExistingObjectTag/Department": "${aws:PrincipalTag/Department}"
        }
      }
    },
    {
      "Sid": "ABACTagResources",
      "Effect": "Allow",
      "Action": [
        "ec2:RunInstances",
        "ec2:CreateVolume"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:RequestTag/Department": "${aws:PrincipalTag/Department}",
          "aws:RequestTag/Project": "${aws:PrincipalTag/Project}"
        }
      }
    }
  ]
}
```

**Tag Users and Resources**
```bash
# Tag user
aws iam tag-user \
  --user-name alice \
  --tags Key=Department,Value=Engineering Key=Project,Value=WebApp

# Tag role
aws iam tag-role \
  --role-name developer-role \
  --tags Key=Department,Value=Engineering Key=Environment,Value=Dev

# User can only access S3 paths matching their department tag
# s3://engineering/* (allowed)
# s3://finance/* (denied)
```

## IAM Roles and Federated Access

### Cross-Account Access

**Trust Relationship**
```json
// In Account B (target account) - Role trust policy
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111111111111:root"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId": "unique-external-id-12345"
        },
        "IpAddress": {
          "aws:SourceIp": "203.0.113.0/24"
        }
      }
    }
  ]
}
```

**Assume Role**
```python
import boto3

sts = boto3.client('sts')

# Assume role in another account
response = sts.assume_role(
    RoleArn='arn:aws:iam::222222222222:role/CrossAccountRole',
    RoleSessionName='session1',
    ExternalId='unique-external-id-12345',
    DurationSeconds=3600
)

# Use temporary credentials
credentials = response['Credentials']
s3 = boto3.client(
    's3',
    aws_access_key_id=credentials['AccessKeyId'],
    aws_secret_access_key=credentials['SecretAccessKey'],
    aws_session_token=credentials['SessionToken']
)
```

### SAML 2.0 Federation

**Enterprise SSO Integration**
```json
// SAML provider trust policy
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::123456789012:saml-provider/MyIDProvider"
      },
      "Action": "sts:AssumeRoleWithSAML",
      "Condition": {
        "StringEquals": {
          "SAML:aud": "https://signin.aws.amazon.com/saml"
        }
      }
    }
  ]
}
```

**Create SAML Provider**
```bash
# Upload IdP metadata
aws iam create-saml-provider \
  --name MyIDProvider \
  --saml-metadata-document file://metadata.xml

# Create role for federated users
aws iam create-role \
  --role-name SAMLFederatedRole \
  --assume-role-policy-document file://saml-trust.json

# Attach permissions
aws iam attach-role-policy \
  --role-name SAMLFederatedRole \
  --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
```

### Web Identity Federation

**OpenID Connect (OIDC)**
```json
// Trust policy for web identity
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::123456789012:oidc-provider/accounts.google.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "accounts.google.com:aud": "app-client-id"
        }
      }
    }
  ]
}
```

**Assume Role with Web Identity**
```python
sts = boto3.client('sts')

response = sts.assume_role_with_web_identity(
    RoleArn='arn:aws:iam::123456789012:role/WebIdentityRole',
    RoleSessionName='web-session',
    WebIdentityToken='<JWT-token-from-IdP>',
    DurationSeconds=3600
)

credentials = response['Credentials']
```

### Service Roles

**EC2 Instance Role**
```yaml
Resources:
  InstanceRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: ec2.amazonaws.com
            Action: 'sts:AssumeRole'
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
      Policies:
        - PolicyName: CloudWatchLogs
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - 'logs:CreateLogGroup'
                  - 'logs:CreateLogStream'
                  - 'logs:PutLogEvents'
                Resource: 'arn:aws:logs:*:*:*'

  InstanceProfile:
    Type: AWS::IAM::InstanceProfile
    Properties:
      Roles:
        - !Ref InstanceRole

  Instance:
    Type: AWS::EC2::Instance
    Properties:
      IamInstanceProfile: !Ref InstanceProfile
      ImageId: ami-12345678
      InstanceType: t3.micro
```

**Lambda Execution Role**
```json
// Trust policy
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}

// Permissions policy
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:Query"
      ],
      "Resource": "arn:aws:dynamodb:region:account:table/MyTable"
    }
  ]
}
```

## IAM Access Analyzer

### Resource Analysis

**Create Access Analyzer**
```bash
# Create analyzer
aws accessanalyzer create-analyzer \
  --analyzer-name organization-analyzer \
  --type ORGANIZATION

# List findings
aws accessanalyzer list-findings \
  --analyzer-arn arn:aws:access-analyzer:region:account:analyzer/organization-analyzer

# Get finding details
aws accessanalyzer get-finding \
  --analyzer-arn arn:aws:access-analyzer:region:account:analyzer/organization-analyzer \
  --finding-id finding-id-12345
```

**Findings Types**
- S3 buckets shared externally
- IAM roles assumable by external accounts
- KMS keys accessible externally
- Lambda functions with external permissions
- SQS queues with cross-account access
- Secrets Manager secrets shared externally

### Unused Access Analysis

**Identify Unused Permissions**
```python
import boto3
from datetime import datetime, timedelta

iam = boto3.client('iam')
analyzer = boto3.client('accessanalyzer')

# Generate service last accessed report
response = iam.generate_service_last_accessed_details(
    Arn='arn:aws:iam::123456789012:role/MyRole'
)

job_id = response['JobId']

# Check status and get results
details = iam.get_service_last_accessed_details(JobId=job_id)

for service in details['ServicesLastAccessed']:
    if 'LastAuthenticated' in service:
        last_used = service['LastAuthenticated']
        days_since = (datetime.now(last_used.tzinfo) - last_used).days
        
        if days_since > 90:
            print(f"{service['ServiceName']}: Not used in {days_since} days")
```

### Policy Generation

**Generate Least-Privilege Policy from CloudTrail**
```bash
# Start policy generation
aws accessanalyzer start-policy-generation \
  --policy-generation-details '{
    "principalArn": "arn:aws:iam::123456789012:role/MyRole",
    "cloudTrailDetails": {
      "trailArn": "arn:aws:cloudtrail:region:account:trail/my-trail",
      "startTime": "2024-01-01T00:00:00Z",
      "endTime": "2024-01-31T23:59:59Z",
      "accessRole": "arn:aws:iam::account:role/AccessAnalyzerRole"
    }
  }'

# Get generated policy
aws accessanalyzer get-generated-policy \
  --job-id job-12345 \
  --include-resource-placeholders
```

## IAM Security Best Practices

### MFA Enforcement

**Require MFA Policy**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyAllExceptListedIfNoMFA",
      "Effect": "Deny",
      "NotAction": [
        "iam:CreateVirtualMFADevice",
        "iam:EnableMFADevice",
        "iam:GetUser",
        "iam:ListMFADevices",
        "iam:ListVirtualMFADevices",
        "iam:ResyncMFADevice",
        "sts:GetSessionToken"
      ],
      "Resource": "*",
      "Condition": {
        "BoolIfExists": {
          "aws:MultiFactorAuthPresent": "false"
        }
      }
    }
  ]
}
```

### Password Policy

**Strong Password Requirements**
```bash
aws iam update-account-password-policy \
  --minimum-password-length 14 \
  --require-symbols \
  --require-numbers \
  --require-uppercase-characters \
  --require-lowercase-characters \
  --allow-users-to-change-password \
  --max-password-age 90 \
  --password-reuse-prevention 24
```

### Credential Rotation

**Access Key Rotation**
```python
import boto3
from datetime import datetime, timedelta

iam = boto3.client('iam')

# List all users
users = iam.list_users()['Users']

for user in users:
    # Get access keys
    keys = iam.list_access_keys(UserName=user['UserName'])
    
    for key in keys['AccessKeyMetadata']:
        created_date = key['CreateDate'].replace(tzinfo=None)
        age = (datetime.now() - created_date).days
        
        if age > 90:
            print(f"WARNING: {user['UserName']} key {key['AccessKeyId']} is {age} days old")
            
            # Optionally rotate
            # new_key = iam.create_access_key(UserName=user['UserName'])
            # iam.update_access_key(
            #     UserName=user['UserName'],
            #     AccessKeyId=key['AccessKeyId'],
            #     Status='Inactive'
            # )
```

### Least Privilege

**Start with Minimum Permissions**
```json
// Initial policy - deny all
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*"
    }
  ]
}

// Add permissions as needed
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::my-specific-bucket"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::my-specific-bucket/*"
    }
  ]
}
```

## Exam Tips

### IAM Concepts
- Users: Long-term credentials
- Roles: Temporary credentials, preferred over users
- Groups: Organize users, simplify management
- Policies: Define permissions

### Policy Evaluation
1. Explicit Deny wins always
2. SCPs apply organization-wide
3. Permission boundaries set maximum
4. Identity policies grant permissions
5. Implicit deny if no explicit allow

### Advanced Features
- Permission Boundaries: Delegate safely
- SCPs: Organization-wide restrictions
- ABAC: Tag-based permissions, scalable
- Access Analyzer: Find external access, unused permissions
- Policy Generator: Create least-privilege policies

### Best Practices
- Enable MFA for all users
- Rotate credentials regularly (90 days)
- Use roles instead of access keys
- Implement least privilege
- Monitor with CloudTrail and Access Analyzer
- Use policy conditions for restrictions
- Tag resources and principals for ABAC

### Common Scenarios
- Cross-account access: Trust policy + assume role
- Federation: SAML 2.0 or OIDC
- Prevent privilege escalation: Permission boundaries
- Organization controls: SCPs
- External access audit: IAM Access Analyzer
