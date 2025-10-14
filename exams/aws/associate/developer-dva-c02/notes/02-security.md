# Domain 2: Security (26%)

## Overview
This domain covers implementing authentication and authorization, encryption, and managing sensitive data in applications. Security is critical for the Developer Associate exam.

## AWS Identity and Access Management (IAM)

**[📖 IAM User Guide](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)** - Complete guide to AWS identity and access management

### IAM Core Concepts

#### IAM Roles for Applications
- **EC2 Instance Roles**: Attach IAM roles to EC2 instances
- **Lambda Execution Roles**: Grant Lambda functions permissions to AWS services
- **ECS Task Roles**: Permissions for containerized applications
- **Cross-Account Roles**: Access resources across AWS accounts

#### IAM Policy Types
```json
// Identity-based policy (attached to users, groups, roles)
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}

// Resource-based policy (attached to resources like S3, SQS, Lambda)
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowAccountAccess",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:root"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

#### IAM Policy Evaluation Logic
1. **Explicit Deny**: Always takes precedence
2. **Explicit Allow**: Permits action if no deny exists
3. **Implicit Deny**: Default deny if no explicit allow
4. **Policy Boundaries**: Sets maximum permissions for IAM entity

**[📖 Policy Evaluation Logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html)** - How AWS evaluates policies to authorize requests

### AWS Security Token Service (STS)

**[📖 AWS STS API Reference](https://docs.aws.amazon.com/STS/latest/APIReference/welcome.html)** - Temporary security credentials with STS

#### Temporary Credentials
```python
import boto3

# Assume role
sts = boto3.client('sts')
response = sts.assume_role(
    RoleArn='arn:aws:iam::123456789012:role/MyRole',
    RoleSessionName='session-name',
    DurationSeconds=3600
)

credentials = response['Credentials']
temp_session = boto3.Session(
    aws_access_key_id=credentials['AccessKeyId'],
    aws_secret_access_key=credentials['SecretAccessKey'],
    aws_session_token=credentials['SessionToken']
)

# Use temporary credentials
s3 = temp_session.client('s3')
```

#### Common STS Operations
- **AssumeRole**: Cross-account or same-account role assumption
- **AssumeRoleWithWebIdentity**: For mobile/web apps with identity providers
- **AssumeRoleWithSAML**: For SAML-based federation
- **GetFederationToken**: For federated users
- **GetSessionToken**: For MFA-enabled users

### IAM Best Practices for Developers

1. **Use IAM Roles**: Never embed credentials in code
2. **Least Privilege**: Grant minimum necessary permissions
3. **Temporary Credentials**: Use STS for temporary access
4. **Policy Conditions**: Use conditions for fine-grained control
5. **Rotate Credentials**: Regular rotation of access keys
6. **Enable MFA**: Multi-factor authentication for sensitive operations

## Amazon Cognito

**[📖 Amazon Cognito Developer Guide](https://docs.aws.amazon.com/cognito/latest/developerguide/what-is-amazon-cognito.html)** - User authentication and authorization for web and mobile apps

### Cognito User Pools

#### User Authentication
```javascript
// AWS Amplify example for User Pool authentication
import { Auth } from 'aws-amplify';

// Sign up
await Auth.signUp({
    username: 'user@example.com',
    password: 'TempPassword123!',
    attributes: {
        email: 'user@example.com',
        name: 'John Doe'
    }
});

// Confirm sign up
await Auth.confirmSignUp('user@example.com', 'confirmation-code');

// Sign in
const user = await Auth.signIn('user@example.com', 'Password123!');

// Get current user
const currentUser = await Auth.currentAuthenticatedUser();

// Sign out
await Auth.signOut();
```

#### User Pool Features
- **Sign-up and Sign-in**: Email/username and password authentication
- **MFA**: SMS or TOTP-based multi-factor authentication
- **Password Policies**: Configurable password requirements
- **Email/SMS Verification**: Account verification workflows
- **Custom Attributes**: Additional user metadata
- **Lambda Triggers**: Pre/post authentication, token generation hooks

### Cognito Identity Pools

**[📖 Amazon Cognito Identity Pools](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-identity.html)** - Federated identities for AWS resource access

#### Federated Access to AWS
```javascript
// Configure identity pool
AWS.config.credentials = new AWS.CognitoIdentityCredentials({
    IdentityPoolId: 'us-east-1:12345678-1234-1234-1234-123456789012',
    Logins: {
        'cognito-idp.us-east-1.amazonaws.com/us-east-1_ABC123': idToken
    }
});

// Get temporary AWS credentials
AWS.config.credentials.get((err) => {
    if (!err) {
        // Use AWS services with temporary credentials
        const s3 = new AWS.S3();
        s3.listBuckets((err, data) => {
            console.log(data.Buckets);
        });
    }
});
```

#### Identity Pool Use Cases
- **Guest Access**: Unauthenticated user access to AWS resources
- **Authenticated Access**: Access after authentication via User Pool or external IdP
- **Role-Based Access**: Different IAM roles for authenticated/unauthenticated users
- **Fine-Grained Access**: Use identity ID in IAM policies for user-specific access

### Cognito Integration Patterns

#### API Gateway + Cognito Authorizer
```yaml
# API Gateway configuration
authorizers:
  CognitoAuthorizer:
    type: COGNITO_USER_POOLS
    providerARNs:
      - arn:aws:cognito-idp:us-east-1:123456789012:userpool/us-east-1_ABC123
    identitySource: method.request.header.Authorization
```

## AWS Key Management Service (KMS)

**[📖 AWS KMS Developer Guide](https://docs.aws.amazon.com/kms/latest/developerguide/overview.html)** - Create and manage encryption keys

### KMS Key Types

#### Customer Managed Keys (CMK)
- **Full Control**: Create, rotate, disable, delete keys
- **Key Policies**: Fine-grained access control
- **Automatic Rotation**: Annual key rotation (optional)
- **CloudTrail Logging**: All key usage logged

#### AWS Managed Keys
- **Service-Specific**: Created by AWS services (e.g., aws/s3, aws/rds)
- **Limited Control**: Cannot delete or modify key policies
- **Automatic Rotation**: Rotated every 3 years

### KMS Operations

#### Encryption and Decryption
```python
import boto3
import base64

kms = boto3.client('kms')

# Encrypt data
plaintext = 'Sensitive data'
response = kms.encrypt(
    KeyId='arn:aws:kms:us-east-1:123456789012:key/12345678-1234-1234-1234-123456789012',
    Plaintext=plaintext
)
ciphertext = base64.b64encode(response['CiphertextBlob']).decode()

# Decrypt data
response = kms.decrypt(
    CiphertextBlob=base64.b64decode(ciphertext)
)
decrypted_plaintext = response['Plaintext'].decode()

# Generate data key (for envelope encryption)
response = kms.generate_data_key(
    KeyId='alias/my-key',
    KeySpec='AES_256'
)
plaintext_key = response['Plaintext']
encrypted_key = response['CiphertextBlob']
```

#### KMS Key Policies
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Enable IAM User Permissions",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:root"
      },
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "Allow use of the key",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/MyAppRole"
      },
      "Action": [
        "kms:Encrypt",
        "kms:Decrypt",
        "kms:GenerateDataKey"
      ],
      "Resource": "*"
    }
  ]
}
```

### Envelope Encryption Pattern

**[📖 Envelope Encryption](https://docs.aws.amazon.com/kms/latest/developerguide/concepts.html#enveloping)** - Learn about envelope encryption with KMS
```python
# 1. Generate data key
response = kms.generate_data_key(KeyId='alias/my-key', KeySpec='AES_256')
plaintext_key = response['Plaintext']
encrypted_key = response['CiphertextBlob']

# 2. Encrypt data with plaintext key
from cryptography.fernet import Fernet
cipher = Fernet(base64.urlsafe_b64encode(plaintext_key[:32]))
encrypted_data = cipher.encrypt(b'Large amount of data')

# 3. Store encrypted data + encrypted key
# 4. To decrypt: first decrypt the data key with KMS, then decrypt data
```

## AWS Secrets Manager

**[📖 AWS Secrets Manager User Guide](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html)** - Manage and rotate secrets throughout their lifecycle

### Managing Application Secrets

#### Store and Retrieve Secrets
```python
import boto3
import json

secrets_client = boto3.client('secretsmanager')

# Create secret
secrets_client.create_secret(
    Name='prod/myapp/db-credentials',
    Description='Database credentials for production',
    SecretString=json.dumps({
        'username': 'admin',
        'password': 'SecurePassword123!',
        'host': 'mydb.cluster-abc.us-east-1.rds.amazonaws.com',
        'port': 3306
    })
)

# Retrieve secret
response = secrets_client.get_secret_value(SecretId='prod/myapp/db-credentials')
secret_dict = json.loads(response['SecretString'])

# Update secret
secrets_client.update_secret(
    SecretId='prod/myapp/db-credentials',
    SecretString=json.dumps({'username': 'admin', 'password': 'NewPassword456!'})
)

# Rotate secret
secrets_client.rotate_secret(
    SecretId='prod/myapp/db-credentials',
    RotationLambdaARN='arn:aws:lambda:us-east-1:123456789012:function:RotateSecret',
    RotationRules={'AutomaticallyAfterDays': 30}
)
```

#### Automatic Secret Rotation
- **Lambda-based Rotation**: Custom Lambda function for rotation logic
- **Rotation Strategies**: Single user vs alternating users
- **RDS Integration**: Built-in rotation for RDS credentials
- **Versioning**: Maintains secret versions during rotation

### Secrets Manager vs Parameter Store

| Feature | Secrets Manager | Parameter Store |
|---------|----------------|-----------------|
| **Purpose** | Secrets rotation and lifecycle | Configuration and secrets |
| **Cost** | $0.40/secret/month + API calls | Free (Standard), $0.05/parameter (Advanced) |
| **Rotation** | Built-in automatic rotation | Manual rotation |
| **Size Limit** | 64 KB | 4 KB (Standard), 8 KB (Advanced) |
| **Use Case** | Database credentials, API keys | Application configuration, simple secrets |

## AWS Systems Manager Parameter Store

**[📖 AWS Systems Manager Parameter Store](https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html)** - Secure storage for configuration data and secrets

### Store Configuration Data
```python
import boto3

ssm = boto3.client('ssm')

# Put parameter
ssm.put_parameter(
    Name='/myapp/config/api-url',
    Value='https://api.example.com',
    Type='String',
    Description='API endpoint URL',
    Tier='Standard'
)

# Put secure parameter
ssm.put_parameter(
    Name='/myapp/config/api-key',
    Value='secret-api-key-12345',
    Type='SecureString',
    KeyId='alias/aws/ssm',  # KMS key for encryption
    Tier='Standard'
)

# Get parameter
response = ssm.get_parameter(Name='/myapp/config/api-url')
value = response['Parameter']['Value']

# Get parameter with decryption
response = ssm.get_parameter(Name='/myapp/config/api-key', WithDecryption=True)
api_key = response['Parameter']['Value']

# Get parameters by path
response = ssm.get_parameters_by_path(
    Path='/myapp/config',
    Recursive=True,
    WithDecryption=True
)
```

## AWS Certificate Manager (ACM)

**[📖 AWS Certificate Manager User Guide](https://docs.aws.amazon.com/acm/latest/userguide/acm-overview.html)** - Provision and manage SSL/TLS certificates

### SSL/TLS Certificate Management

#### Request Certificate
```bash
# Request public certificate
aws acm request-certificate \
    --domain-name example.com \
    --subject-alternative-names www.example.com \
    --validation-method DNS

# Import certificate
aws acm import-certificate \
    --certificate fileb://certificate.pem \
    --private-key fileb://private-key.pem \
    --certificate-chain fileb://certificate-chain.pem
```

#### ACM Integration
- **Load Balancers**: ALB/NLB SSL termination
- **CloudFront**: HTTPS for CDN distributions
- **API Gateway**: Custom domain SSL/TLS
- **Elastic Beanstalk**: SSL for applications
- **Automatic Renewal**: ACM auto-renews public certificates

## Application Security Best Practices

### Secure Coding Practices

#### Input Validation
```python
import re
from html import escape

def validate_email(email):
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return re.match(pattern, email) is not None

def sanitize_input(user_input):
    # Escape HTML to prevent XSS
    return escape(user_input)

def validate_sql_input(user_input):
    # Use parameterized queries instead of string concatenation
    # GOOD: cursor.execute("SELECT * FROM users WHERE id = %s", (user_id,))
    # BAD:  cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")
    pass
```

#### Secure Environment Variables
```python
import os

# Load sensitive data from environment variables
DB_HOST = os.environ.get('DB_HOST')
DB_PASSWORD = os.environ.get('DB_PASSWORD')
API_KEY = os.environ.get('API_KEY')

# For Lambda, load from Secrets Manager
def get_db_credentials():
    secret_name = os.environ['SECRET_NAME']
    response = secrets_client.get_secret_value(SecretId=secret_name)
    return json.loads(response['SecretString'])
```

### Encryption Best Practices

**[📖 Data Protection in AWS](https://docs.aws.amazon.com/whitepapers/latest/logical-separation/encrypting-data-at-rest-and--in-transit.html)** - Best practices for encrypting data at rest and in transit

#### Data at Rest
- **S3**: Server-side encryption (SSE-S3, SSE-KMS, SSE-C)
- **EBS**: Encrypted volumes with KMS
- **RDS**: Encryption with KMS at creation time
- **DynamoDB**: Encryption at rest enabled by default

#### Data in Transit
- **HTTPS**: All API communication over TLS
- **VPC Endpoints**: Private connectivity to AWS services
- **VPN**: Encrypted connections to VPC
- **TLS 1.2+**: Use modern TLS versions

### IAM Permission Boundaries

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:*",
        "dynamodb:*",
        "lambda:*"
      ],
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

## Common Security Scenarios

### Scenario 1: Secure API with Cognito
```
Client → API Gateway (Cognito Authorizer) → Lambda → DynamoDB
- User authenticates with Cognito User Pool
- Receives JWT token
- Token validated by API Gateway
- Lambda accesses DynamoDB with IAM role
```

### Scenario 2: Encrypt Sensitive Data
```
Application → KMS (generate data key) → Encrypt data → S3 (encrypted object)
- Generate data key from KMS
- Encrypt data with data key (envelope encryption)
- Store encrypted data and encrypted key
- Decrypt: retrieve encrypted key, decrypt with KMS, decrypt data
```

### Scenario 3: Database Credential Rotation
```
Application → Secrets Manager → RDS
- Application retrieves credentials from Secrets Manager
- Lambda function rotates credentials automatically
- Application always uses current credentials
- Zero downtime rotation
```

## Study Tips

1. **IAM Mastery**: Understand policy evaluation, roles, and temporary credentials
2. **Cognito Deep Dive**: Know User Pools vs Identity Pools use cases
3. **KMS Operations**: Practice encryption, decryption, and envelope encryption
4. **Secrets Management**: Compare Secrets Manager vs Parameter Store
5. **Certificate Management**: Understand ACM integration with AWS services
6. **Security Best Practices**: Always choose most secure option in exam scenarios

## CLI Quick Reference

```bash
# IAM
aws iam create-role --role-name MyRole --assume-role-policy-document file://trust-policy.json
aws iam attach-role-policy --role-name MyRole --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
aws sts assume-role --role-arn arn:aws:iam::123456789012:role/MyRole --role-session-name session1

# KMS
aws kms create-key --description "My application key"
aws kms encrypt --key-id alias/my-key --plaintext "secret data"
aws kms decrypt --ciphertext-blob fileb://encrypted.dat

# Secrets Manager
aws secretsmanager create-secret --name MySecret --secret-string "MyPassword"
aws secretsmanager get-secret-value --secret-id MySecret
aws secretsmanager rotate-secret --secret-id MySecret

# Parameter Store
aws ssm put-parameter --name /myapp/db/password --value "password" --type SecureString
aws ssm get-parameter --name /myapp/db/password --with-decryption
aws ssm get-parameters-by-path --path /myapp --recursive
```
