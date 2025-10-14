# Domain 3: Deployment (24%)

## Overview
This domain covers preparing application artifacts, testing in development environments, automating deployment, and deploying code using AWS CI/CD services. Understanding deployment strategies and automation is crucial for developers.

## AWS Developer Tools

### AWS CodeCommit

**[📖 AWS CodeCommit User Guide](https://docs.aws.amazon.com/codecommit/latest/userguide/welcome.html)** - Secure Git repositories hosted on AWS

#### Git Repository Service
- **Fully Managed**: No infrastructure to manage
- **High Availability**: Redundant across multiple AZs
- **Encryption**: At rest and in transit
- **IAM Integration**: Fine-grained access control
- **Event Notifications**: Triggers for repository events

#### CodeCommit Operations
```bash
# Clone repository
git clone https://git-codecommit.us-east-1.amazonaws.com/v1/repos/MyRepo

# Configure Git credentials helper
git config --global credential.helper '!aws codecommit credential-helper $@'
git config --global credential.UseHttpPath true

# Push changes
git add .
git commit -m "Update application code"
git push origin main

# Create branch
git checkout -b feature-branch
git push origin feature-branch

# Merge via pull request (AWS CLI)
aws codecommit create-pull-request \
    --title "New Feature" \
    --description "Description of changes" \
    --targets repositoryName=MyRepo,sourceReference=feature-branch,destinationReference=main
```

#### CodeCommit Triggers
```json
{
  "triggers": [
    {
      "name": "NotifyOnPush",
      "destinationArn": "arn:aws:sns:us-east-1:123456789012:MyTopic",
      "events": ["all"]
    },
    {
      "name": "InvokeLambda",
      "destinationArn": "arn:aws:lambda:us-east-1:123456789012:function:MyFunction",
      "events": ["updateReference"]
    }
  ]
}
```

### AWS CodeBuild

**[📖 AWS CodeBuild User Guide](https://docs.aws.amazon.com/codebuild/latest/userguide/welcome.html)** - Fully managed continuous integration service

#### Build Service Features
- **Managed Build Environment**: Preconfigured build environments
- **Custom Docker Images**: Use custom build environments
- **Parallel Builds**: Build multiple projects simultaneously
- **Build Caching**: Cache dependencies for faster builds
- **Artifact Management**: Store build outputs in S3
- **Environment Variables**: Secure parameter storage

#### buildspec.yml

**[📖 BuildSpec File Reference](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html)** - Complete buildspec syntax reference
```yaml
version: 0.2

env:
  variables:
    NODE_ENV: production
  parameter-store:
    DB_PASSWORD: /myapp/db/password
  secrets-manager:
    API_KEY: prod/myapp/api:key

phases:
  install:
    runtime-versions:
      nodejs: 14
    commands:
      - echo "Installing dependencies"
      - npm install

  pre_build:
    commands:
      - echo "Running tests"
      - npm test
      - echo "Linting code"
      - npm run lint

  build:
    commands:
      - echo "Building application"
      - npm run build

  post_build:
    commands:
      - echo "Build completed on `date`"

artifacts:
  files:
    - '**/*'
  base-directory: dist
  name: BuildArtifact-$(date +%Y%m%d-%H%M%S)

cache:
  paths:
    - 'node_modules/**/*'
```

#### Advanced buildspec Features
```yaml
# Multi-phase builds
phases:
  install:
    commands:
      - apt-get update
      - apt-get install -y python3

  pre_build:
    commands:
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $ECR_REPOSITORY
      - COMMIT_HASH=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
      - IMAGE_TAG=${COMMIT_HASH:=latest}

  build:
    commands:
      - docker build -t $ECR_REPOSITORY:$IMAGE_TAG .
      - docker tag $ECR_REPOSITORY:$IMAGE_TAG $ECR_REPOSITORY:latest

  post_build:
    commands:
      - docker push $ECR_REPOSITORY:$IMAGE_TAG
      - docker push $ECR_REPOSITORY:latest
      - echo "[{\"name\":\"myapp\",\"imageUri\":\"$ECR_REPOSITORY:$IMAGE_TAG\"}]" > imagedefinitions.json

artifacts:
  files:
    - imagedefinitions.json

reports:
  test-reports:
    files:
      - 'test-results/**/*'
    file-format: JUNITXML
```

### AWS CodeDeploy

**[📖 AWS CodeDeploy User Guide](https://docs.aws.amazon.com/codedeploy/latest/userguide/welcome.html)** - Automate code deployments to maintain application uptime

#### Deployment Platforms
- **EC2/On-Premises**: Traditional server deployments
- **Lambda**: Serverless function deployments
- **ECS**: Container deployments

#### Deployment Configurations

##### EC2 Deployment Types
```yaml
# In-place deployment (rolling update)
version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/html
hooks:
  BeforeInstall:
    - location: scripts/install_dependencies.sh
      timeout: 300
      runas: root
  ApplicationStart:
    - location: scripts/start_server.sh
      timeout: 300
      runas: root
  ValidateService:
    - location: scripts/validate_service.sh
      timeout: 300
```

##### Lambda Deployment (appspec.yml)
```yaml
version: 0.0
Resources:
  - MyLambdaFunction:
      Type: AWS::Lambda::Function
      Properties:
        Name: "myfunction"
        Alias: "prod"
        CurrentVersion: "1"
        TargetVersion: "2"
Hooks:
  - BeforeAllowTraffic: "ValidateBeforeTraffic"
  - AfterAllowTraffic: "ValidateAfterTraffic"
```

#### Deployment Strategies

**[📖 Deployment Configurations](https://docs.aws.amazon.com/codedeploy/latest/userguide/deployment-configurations.html)** - Learn about deployment strategies and configurations

##### Blue/Green Deployment
- **Traffic Shifting**: Gradual shift from old to new version
- **Canary**: 10% traffic, then 100% if successful
- **Linear**: Gradually increase traffic in steps
- **All-at-once**: Immediate switch to new version

```bash
# Create deployment for Lambda
aws deploy create-deployment \
    --application-name MyApp \
    --deployment-group-name Production \
    --deployment-config-name CodeDeployDefault.LambdaCanary10Percent5Minutes \
    --description "Deploy new Lambda version"
```

##### Rolling Deployment Configurations
- **OneAtATime**: Deploy to one instance at a time
- **HalfAtATime**: Deploy to 50% of instances
- **AllAtOnce**: Deploy to all instances simultaneously
- **Custom**: Define custom percentage or count

### AWS CodePipeline

**[📖 AWS CodePipeline User Guide](https://docs.aws.amazon.com/codepipeline/latest/userguide/welcome.html)** - Automate continuous delivery pipelines

#### Pipeline Structure
```json
{
  "pipeline": {
    "name": "MyPipeline",
    "roleArn": "arn:aws:iam::123456789012:role/CodePipelineServiceRole",
    "artifactStore": {
      "type": "S3",
      "location": "my-pipeline-artifacts"
    },
    "stages": [
      {
        "name": "Source",
        "actions": [
          {
            "name": "SourceAction",
            "actionTypeId": {
              "category": "Source",
              "owner": "AWS",
              "provider": "CodeCommit",
              "version": "1"
            },
            "configuration": {
              "RepositoryName": "MyRepo",
              "BranchName": "main"
            },
            "outputArtifacts": [{"name": "SourceOutput"}]
          }
        ]
      },
      {
        "name": "Build",
        "actions": [
          {
            "name": "BuildAction",
            "actionTypeId": {
              "category": "Build",
              "owner": "AWS",
              "provider": "CodeBuild",
              "version": "1"
            },
            "configuration": {
              "ProjectName": "MyBuildProject"
            },
            "inputArtifacts": [{"name": "SourceOutput"}],
            "outputArtifacts": [{"name": "BuildOutput"}]
          }
        ]
      },
      {
        "name": "Deploy",
        "actions": [
          {
            "name": "DeployAction",
            "actionTypeId": {
              "category": "Deploy",
              "owner": "AWS",
              "provider": "CodeDeploy",
              "version": "1"
            },
            "configuration": {
              "ApplicationName": "MyApp",
              "DeploymentGroupName": "Production"
            },
            "inputArtifacts": [{"name": "BuildOutput"}]
          }
        ]
      }
    ]
  }
}
```

#### Advanced Pipeline Features

##### Parallel Actions
```yaml
stages:
  - name: Test
    actions:
      - name: UnitTests
        actionTypeId:
          category: Test
          provider: CodeBuild
        runOrder: 1
      - name: IntegrationTests
        actionTypeId:
          category: Test
          provider: CodeBuild
        runOrder: 1  # Same runOrder = parallel execution
```

##### Manual Approval
```json
{
  "name": "ApprovalStage",
  "actions": [
    {
      "name": "ManualApproval",
      "actionTypeId": {
        "category": "Approval",
        "owner": "AWS",
        "provider": "Manual",
        "version": "1"
      },
      "configuration": {
        "CustomData": "Please review and approve deployment to production",
        "NotificationArn": "arn:aws:sns:us-east-1:123456789012:ApprovalTopic"
      }
    }
  ]
}
```

## AWS CloudFormation

**[📖 AWS CloudFormation User Guide](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)** - Model and provision AWS resources using infrastructure as code

### Infrastructure as Code

#### CloudFormation Template Structure
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Sample application infrastructure

Parameters:
  EnvironmentName:
    Type: String
    Default: dev
    AllowedValues:
      - dev
      - test
      - prod

  InstanceType:
    Type: String
    Default: t3.micro
    Description: EC2 instance type

Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-0c55b159cbfafe1f0
    us-west-2:
      AMI: ami-0d1cd67c26f5fca19

Conditions:
  IsProduction: !Equals [!Ref EnvironmentName, prod]

Resources:
  MySecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow HTTP and SSH
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0

  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !FindInMap [RegionMap, !Ref 'AWS::Region', AMI]
      InstanceType: !Ref InstanceType
      SecurityGroupIds:
        - !Ref MySecurityGroup
      Tags:
        - Key: Name
          Value: !Sub '${EnvironmentName}-server'

  MyBucket:
    Type: AWS::S3::Bucket
    Condition: IsProduction
    Properties:
      BucketName: !Sub '${EnvironmentName}-data-bucket'
      VersioningConfiguration:
        Status: Enabled

Outputs:
  InstanceId:
    Description: EC2 Instance ID
    Value: !Ref MyEC2Instance
    Export:
      Name: !Sub '${AWS::StackName}-InstanceId'

  BucketName:
    Description: S3 Bucket Name
    Value: !Ref MyBucket
    Condition: IsProduction
```

#### CloudFormation Operations
```bash
# Create stack
aws cloudformation create-stack \
    --stack-name my-app-stack \
    --template-body file://template.yaml \
    --parameters ParameterKey=EnvironmentName,ParameterValue=prod \
    --capabilities CAPABILITY_IAM

# Update stack
aws cloudformation update-stack \
    --stack-name my-app-stack \
    --template-body file://template-updated.yaml

# Delete stack
aws cloudformation delete-stack --stack-name my-app-stack

# Describe stack events
aws cloudformation describe-stack-events --stack-name my-app-stack

# Detect drift
aws cloudformation detect-stack-drift --stack-name my-app-stack
```

#### Nested Stacks

**[📖 Nested Stacks](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-nested-stacks.html)** - Create reusable templates with nested stacks
```yaml
Resources:
  NetworkStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/mybucket/network-template.yaml
      Parameters:
        VPCCidr: 10.0.0.0/16

  ApplicationStack:
    Type: AWS::CloudFormation::Stack
    DependsOn: NetworkStack
    Properties:
      TemplateURL: https://s3.amazonaws.com/mybucket/app-template.yaml
      Parameters:
        VPCId: !GetAtt NetworkStack.Outputs.VPCId
```

## AWS Elastic Beanstalk

**[📖 AWS Elastic Beanstalk Developer Guide](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/Welcome.html)** - Deploy and manage applications without worrying about infrastructure

### Platform as a Service (PaaS)

#### Supported Platforms
- **Languages**: Java, .NET, PHP, Node.js, Python, Ruby, Go
- **Containers**: Docker (single/multi-container)
- **Custom**: Custom platform via Packer

#### Elastic Beanstalk Configuration

##### .ebextensions Configuration
```yaml
# .ebextensions/01-environment.config
option_settings:
  aws:elasticbeanstalk:application:environment:
    DB_HOST: mydb.cluster-abc.us-east-1.rds.amazonaws.com
    DB_NAME: myappdb
    NODE_ENV: production

  aws:autoscaling:launchconfiguration:
    InstanceType: t3.small
    IamInstanceProfile: aws-elasticbeanstalk-ec2-role

  aws:autoscaling:asg:
    MinSize: 2
    MaxSize: 8

  aws:elasticbeanstalk:environment:
    LoadBalancerType: application
```

```yaml
# .ebextensions/02-packages.config
packages:
  yum:
    git: []
    htop: []

commands:
  01_install_dependencies:
    command: "npm install --production"
    cwd: "/var/app/current"
```

#### Deployment Policies
```bash
# All at once (downtime)
eb deploy --label v1.0

# Rolling (no downtime, reduced capacity)
# Configure in .ebextensions or console
option_settings:
  aws:elasticbeanstalk:command:
    DeploymentPolicy: Rolling
    BatchSizeType: Percentage
    BatchSize: 30

# Rolling with additional batch (no reduced capacity)
option_settings:
  aws:elasticbeanstalk:command:
    DeploymentPolicy: RollingWithAdditionalBatch
    BatchSize: 2

# Immutable (safest, creates new instances)
option_settings:
  aws:elasticbeanstalk:command:
    DeploymentPolicy: Immutable

# Blue/Green (manual via environment swap)
eb swap my-env-blue --destination-name my-env-green
```

#### Elastic Beanstalk CLI
```bash
# Initialize application
eb init -p python-3.8 my-app --region us-east-1

# Create environment
eb create my-env --instance-type t3.micro --single

# Deploy application
eb deploy

# View logs
eb logs --stream

# SSH to instance
eb ssh

# Environment status
eb status

# Set environment variables
eb setenv DB_HOST=mydb.example.com DB_NAME=myapp

# Scale environment
eb scale 4
```

## Lambda Deployment Best Practices

### Versioning and Aliases

#### Lambda Versions
```bash
# Publish new version
aws lambda publish-version --function-name my-function

# Create alias pointing to version
aws lambda create-alias \
    --function-name my-function \
    --name prod \
    --function-version 3

# Update alias (traffic shifting)
aws lambda update-alias \
    --function-name my-function \
    --name prod \
    --function-version 4 \
    --routing-config AdditionalVersionWeights={"3"=0.9}  # 90% v3, 10% v4
```

### Lambda Deployment with SAM

**[📖 AWS SAM Developer Guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)** - Build serverless applications with AWS SAM

#### SAM Template
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: SAM template for serverless application

Globals:
  Function:
    Timeout: 30
    Runtime: python3.9
    Environment:
      Variables:
        TABLE_NAME: !Ref MyDynamoDBTable

Resources:
  MyLambdaFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: lambda/
      Handler: app.lambda_handler
      Events:
        ApiEvent:
          Type: Api
          Properties:
            Path: /items
            Method: get
      Policies:
        - DynamoDBCrudPolicy:
            TableName: !Ref MyDynamoDBTable

  MyDynamoDBTable:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: Items
      BillingMode: PAY_PER_REQUEST
      AttributeDefinitions:
        - AttributeName: id
          AttributeType: S
      KeySchema:
        - AttributeName: id
          KeyType: HASH

Outputs:
  ApiEndpoint:
    Description: API Gateway endpoint URL
    Value: !Sub "https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/Prod"
```

#### SAM CLI Commands
```bash
# Build application
sam build

# Test locally
sam local start-api
sam local invoke MyLambdaFunction -e events/event.json

# Deploy application
sam deploy --guided

# View logs
sam logs -n MyLambdaFunction --tail
```

## Container Deployment

### Amazon ECS Deployment

**[📖 Amazon ECS Developer Guide](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)** - Run and manage Docker containers on AWS

#### Task Definition
```json
{
  "family": "myapp",
  "containerDefinitions": [
    {
      "name": "app",
      "image": "123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest",
      "memory": 512,
      "cpu": 256,
      "essential": true,
      "portMappings": [
        {
          "containerPort": 8080,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {"name": "NODE_ENV", "value": "production"}
      ],
      "secrets": [
        {
          "name": "DB_PASSWORD",
          "valueFrom": "arn:aws:secretsmanager:us-east-1:123456789012:secret:db-password"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/myapp",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ],
  "requiresCompatibilities": ["FARGATE"],
  "networkMode": "awsvpc",
  "cpu": "256",
  "memory": "512"
}
```

## Testing in Development Environments

### Local Testing Strategies

#### Lambda Local Testing
```bash
# SAM Local
sam local invoke MyFunction -e event.json

# AWS SAM CLI test with API Gateway
sam local start-api

# Docker-based Lambda testing
docker run -p 9000:8080 amazon/aws-lambda-nodejs:14 app.handler
```

#### Integration Testing
```python
import boto3
import pytest
from moto import mock_dynamodb, mock_s3

@mock_dynamodb
def test_dynamodb_operations():
    # Create mock DynamoDB table
    dynamodb = boto3.resource('dynamodb', region_name='us-east-1')
    table = dynamodb.create_table(
        TableName='Users',
        KeySchema=[{'AttributeName': 'id', 'KeyType': 'HASH'}],
        AttributeDefinitions=[{'AttributeName': 'id', 'AttributeType': 'S'}],
        BillingMode='PAY_PER_REQUEST'
    )

    # Test application code
    table.put_item(Item={'id': '123', 'name': 'Test User'})
    response = table.get_item(Key={'id': '123'})
    assert response['Item']['name'] == 'Test User'

@mock_s3
def test_s3_operations():
    s3 = boto3.client('s3', region_name='us-east-1')
    s3.create_bucket(Bucket='test-bucket')
    s3.put_object(Bucket='test-bucket', Key='test.txt', Body=b'test data')

    response = s3.get_object(Bucket='test-bucket', Key='test.txt')
    assert response['Body'].read() == b'test data'
```

## Study Tips

1. **CI/CD Pipeline**: Build complete pipeline with CodeCommit, CodeBuild, CodeDeploy, CodePipeline
2. **Deployment Strategies**: Understand blue/green, rolling, and canary deployments
3. **CloudFormation**: Practice writing templates with parameters, conditions, and outputs
4. **Elastic Beanstalk**: Know deployment policies and configuration files
5. **Lambda Deployment**: Understand versions, aliases, and traffic shifting
6. **Container Deployment**: ECS task definitions and deployment strategies

## Common Exam Scenarios

- Setting up CI/CD pipeline for serverless applications
- Implementing blue/green deployment for zero downtime
- Automating infrastructure deployment with CloudFormation
- Managing multiple environments (dev/test/prod) with Elastic Beanstalk
- Lambda deployment with gradual traffic shifting
- Container deployment strategies with ECS
- Automated testing in build pipelines

## CLI Quick Reference

```bash
# CodeCommit
aws codecommit create-repository --repository-name MyRepo
aws codecommit list-repositories

# CodeBuild
aws codebuild start-build --project-name MyProject
aws codebuild batch-get-builds --ids build-id

# CodeDeploy
aws deploy create-deployment --application-name MyApp --deployment-group-name Production
aws deploy get-deployment --deployment-id deployment-id

# CodePipeline
aws codepipeline create-pipeline --cli-input-json file://pipeline.json
aws codepipeline start-pipeline-execution --name MyPipeline

# CloudFormation
aws cloudformation create-stack --stack-name MyStack --template-body file://template.yaml
aws cloudformation update-stack --stack-name MyStack --template-body file://template.yaml
aws cloudformation describe-stacks --stack-name MyStack

# Elastic Beanstalk
eb init
eb create my-environment
eb deploy
eb terminate my-environment
```
