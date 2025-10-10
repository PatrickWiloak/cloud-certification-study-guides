# CI/CD Pipelines and Deployment Strategies

## CI/CD Fundamentals

### Continuous Integration (CI)

**Core Principles**
- Frequent code commits to shared repository
- Automated build and test on every commit
- Fast feedback on code quality
- Early detection of integration issues
- Maintain always-deployable main branch

**CI Best Practices**
- Commit code frequently (multiple times per day)
- Automated testing at multiple levels
- Fast build times (under 10 minutes)
- Fix broken builds immediately
- Keep builds and tests deterministic
- Version control everything (code, configs, infrastructure)

### Continuous Delivery vs Continuous Deployment

**Continuous Delivery**
- Every change is deployable
- Manual approval for production deployment
- Automated deployment to staging/test environments
- Business decides when to release

**Continuous Deployment**
- Fully automated deployment to production
- No manual intervention
- Every passing change goes to production
- Requires mature testing and monitoring

## AWS CodePipeline

### Pipeline Architecture

**Core Components**
- Source: CodeCommit, GitHub, S3, ECR
- Build: CodeBuild, Jenkins
- Test: CodeBuild, third-party tools
- Deploy: CodeDeploy, CloudFormation, ECS, Lambda
- Approval: Manual approval actions

**Basic Pipeline Structure**
```yaml
# CloudFormation template for CodePipeline
Resources:
  Pipeline:
    Type: AWS::CodePipeline::Pipeline
    Properties:
      RoleArn: !GetAtt PipelineRole.Arn
      ArtifactStore:
        Type: S3
        Location: !Ref ArtifactBucket
      Stages:
        - Name: Source
          Actions:
            - Name: SourceAction
              ActionTypeId:
                Category: Source
                Owner: AWS
                Provider: CodeCommit
                Version: '1'
              Configuration:
                RepositoryName: !Ref Repository
                BranchName: main
              OutputArtifacts:
                - Name: SourceOutput

        - Name: Build
          Actions:
            - Name: BuildAction
              ActionTypeId:
                Category: Build
                Owner: AWS
                Provider: CodeBuild
                Version: '1'
              Configuration:
                ProjectName: !Ref BuildProject
              InputArtifacts:
                - Name: SourceOutput
              OutputArtifacts:
                - Name: BuildOutput

        - Name: Test
          Actions:
            - Name: UnitTest
              ActionTypeId:
                Category: Test
                Owner: AWS
                Provider: CodeBuild
                Version: '1'
              Configuration:
                ProjectName: !Ref TestProject
              InputArtifacts:
                - Name: BuildOutput

        - Name: Approval
          Actions:
            - Name: ManualApproval
              ActionTypeId:
                Category: Approval
                Owner: AWS
                Provider: Manual
                Version: '1'
              Configuration:
                CustomData: 'Please approve deployment to production'
                NotificationArn: !Ref ApprovalTopic

        - Name: Deploy
          Actions:
            - Name: DeployAction
              ActionTypeId:
                Category: Deploy
                Owner: AWS
                Provider: CodeDeploy
                Version: '1'
              Configuration:
                ApplicationName: !Ref Application
                DeploymentGroupName: !Ref DeploymentGroup
              InputArtifacts:
                - Name: BuildOutput
```

### Advanced Pipeline Patterns

**Parallel Actions**
```yaml
- Name: ParallelTesting
  Actions:
    - Name: UnitTests
      RunOrder: 1
      ActionTypeId:
        Category: Test
        Owner: AWS
        Provider: CodeBuild
        Version: '1'
      Configuration:
        ProjectName: !Ref UnitTestProject

    - Name: IntegrationTests
      RunOrder: 1  # Same RunOrder = parallel execution
      ActionTypeId:
        Category: Test
        Owner: AWS
        Provider: CodeBuild
        Version: '1'
      Configuration:
        ProjectName: !Ref IntegrationTestProject

    - Name: SecurityScan
      RunOrder: 1
      ActionTypeId:
        Category: Test
        Owner: AWS
        Provider: CodeBuild
        Version: '1'
      Configuration:
        ProjectName: !Ref SecurityScanProject
```

**Multi-Region Deployment**
```yaml
- Name: DeployProduction
  Actions:
    - Name: DeployUSEast1
      Region: us-east-1
      ActionTypeId:
        Category: Deploy
        Owner: AWS
        Provider: CloudFormation
        Version: '1'
      Configuration:
        ActionMode: CREATE_UPDATE
        StackName: app-stack
        TemplatePath: BuildOutput::template.yaml

    - Name: DeployUSWest2
      Region: us-west-2
      ActionTypeId:
        Category: Deploy
        Owner: AWS
        Provider: CloudFormation
        Version: '1'
      Configuration:
        ActionMode: CREATE_UPDATE
        StackName: app-stack
        TemplatePath: BuildOutput::template.yaml
```

**Cross-Account Deployment**
```yaml
# In target account, create role for deployment
Resources:
  CrossAccountRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              AWS: !Sub 'arn:aws:iam::${ToolsAccountId}:root'
            Action: 'sts:AssumeRole'
      ManagedPolicyArns:
        - 'arn:aws:iam::aws:policy/PowerUserAccess'

# In pipeline account
- Name: DeployToProduction
  Actions:
    - Name: DeployAction
      RoleArn: !Sub 'arn:aws:iam::${ProductionAccountId}:role/CrossAccountRole'
      ActionTypeId:
        Category: Deploy
        Owner: AWS
        Provider: CloudFormation
        Version: '1'
```

## AWS CodeBuild

### Build Specification

**buildspec.yml Structure**
```yaml
version: 0.2

# Environment variables
env:
  variables:
    NODE_ENV: production
  parameter-store:
    DB_PASSWORD: /prod/db/password
  secrets-manager:
    API_KEY: prod/api:key

# Build phases
phases:
  install:
    runtime-versions:
      nodejs: 18
      python: 3.11
    commands:
      - npm install -g npm@latest
      - pip install -r requirements.txt

  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
      - COMMIT_HASH=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
      - IMAGE_TAG=${COMMIT_HASH:=latest}

  build:
    commands:
      - echo Build started on `date`
      - npm run test
      - npm run build
      - docker build -t $IMAGE_REPO_NAME:$IMAGE_TAG .
      - docker tag $IMAGE_REPO_NAME:$IMAGE_TAG $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG

  post_build:
    commands:
      - echo Build completed on `date`
      - docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG
      - printf '[{"name":"app","imageUri":"%s"}]' $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG > imagedefinitions.json

# Artifacts to pass to next stage
artifacts:
  files:
    - imagedefinitions.json
    - '**/*'
  name: BuildArtifact

# Reports for test results and code coverage
reports:
  jest_reports:
    files:
      - 'test-results.xml'
    file-format: 'JUNITXML'
  coverage_report:
    files:
      - 'coverage/clover.xml'
    file-format: 'CLOVERXML'

# Cache dependencies for faster builds
cache:
  paths:
    - 'node_modules/**/*'
    - '.npm/**/*'
```

### Build Project Configuration

**Advanced Build Settings**
```yaml
Resources:
  BuildProject:
    Type: AWS::CodeBuild::Project
    Properties:
      Name: my-build-project
      ServiceRole: !GetAtt BuildRole.Arn
      Artifacts:
        Type: CODEPIPELINE
      Environment:
        Type: LINUX_CONTAINER
        ComputeType: BUILD_GENERAL1_MEDIUM
        Image: aws/codebuild/standard:7.0
        PrivilegedMode: true  # Required for Docker builds
        EnvironmentVariables:
          - Name: AWS_ACCOUNT_ID
            Value: !Ref AWS::AccountId
          - Name: IMAGE_REPO_NAME
            Value: !Ref ECRRepository
      Source:
        Type: CODEPIPELINE
        BuildSpec: buildspec.yml
      Cache:
        Type: S3
        Location: !Sub '${CacheBucket}/cache'
      LogsConfig:
        CloudWatchLogs:
          Status: ENABLED
          GroupName: /aws/codebuild/my-project
        S3Logs:
          Status: ENABLED
          Location: !Sub '${LogsBucket}/build-logs'
      VpcConfig:  # Build in VPC for private resource access
        VpcId: !Ref VPC
        Subnets:
          - !Ref PrivateSubnet1
          - !Ref PrivateSubnet2
        SecurityGroupIds:
          - !Ref BuildSecurityGroup
```

**Build with Multiple Buildspecs**
```yaml
# Primary build
- Name: Build
  Actions:
    - Name: BuildApp
      ActionTypeId:
        Category: Build
        Owner: AWS
        Provider: CodeBuild
        Version: '1'
      Configuration:
        ProjectName: !Ref BuildProject
        PrimarySource: SourceOutput
        EnvironmentVariables: |
          [
            {"name":"BUILD_TYPE","value":"application","type":"PLAINTEXT"}
          ]

# Secondary build for different artifact
- Name: BuildDocumentation
  Actions:
    - Name: BuildDocs
      ActionTypeId:
        Category: Build
        Owner: AWS
        Provider: CodeBuild
        Version: '1'
      Configuration:
        ProjectName: !Ref DocsBuildProject
        EnvironmentVariables: |
          [
            {"name":"BUILD_TYPE","value":"documentation","type":"PLAINTEXT"}
          ]
```

## AWS CodeDeploy

### Deployment Configurations

**EC2/On-Premises Deployments**

**Application Specification (appspec.yml)**
```yaml
version: 0.0
os: linux

files:
  - source: /
    destination: /var/www/html

permissions:
  - object: /var/www/html
    owner: www-data
    group: www-data
    mode: 755
    type:
      - directory
  - object: /var/www/html
    owner: www-data
    group: www-data
    mode: 644
    type:
      - file

hooks:
  ApplicationStop:
    - location: scripts/stop_application.sh
      timeout: 300
      runas: root

  BeforeInstall:
    - location: scripts/install_dependencies.sh
      timeout: 300
      runas: root

  AfterInstall:
    - location: scripts/configure_application.sh
      timeout: 300
      runas: root

  ApplicationStart:
    - location: scripts/start_application.sh
      timeout: 300
      runas: root

  ValidateService:
    - location: scripts/validate_service.sh
      timeout: 300
      runas: root
```

**Deployment Group Configuration**
```yaml
Resources:
  DeploymentGroup:
    Type: AWS::CodeDeploy::DeploymentGroup
    Properties:
      ApplicationName: !Ref Application
      DeploymentGroupName: production
      ServiceRoleArn: !GetAtt CodeDeployRole.Arn
      DeploymentConfigName: CodeDeployDefault.HalfAtATime
      Ec2TagFilters:
        - Type: KEY_AND_VALUE
          Key: Environment
          Value: production
      AutoRollbackConfiguration:
        Enabled: true
        Events:
          - DEPLOYMENT_FAILURE
          - DEPLOYMENT_STOP_ON_ALARM
      AlarmConfiguration:
        Enabled: true
        Alarms:
          - Name: !Ref HighErrorRateAlarm
      LoadBalancerInfo:
        TargetGroupInfoList:
          - Name: !GetAtt TargetGroup.TargetGroupName
```

**ECS Deployments**

**ECS appspec.yaml**
```yaml
version: 0.0
Resources:
  - TargetService:
      Type: AWS::ECS::Service
      Properties:
        TaskDefinition: <TASK_DEFINITION>
        LoadBalancerInfo:
          ContainerName: "app"
          ContainerPort: 80
        PlatformVersion: "LATEST"
        NetworkConfiguration:
          AwsvpcConfiguration:
            Subnets:
              - "subnet-12345"
              - "subnet-67890"
            SecurityGroups:
              - "sg-12345"
            AssignPublicIp: "DISABLED"

Hooks:
  - BeforeInstall: "LambdaFunctionToValidateBeforeInstall"
  - AfterInstall: "LambdaFunctionToValidateAfterInstall"
  - AfterAllowTestTraffic: "LambdaFunctionToValidateAfterTestTraffic"
  - BeforeAllowTraffic: "LambdaFunctionToValidateBeforeTraffic"
  - AfterAllowTraffic: "LambdaFunctionToValidateAfterTraffic"
```

**Lambda Deployments**

**Lambda appspec.yaml**
```yaml
version: 0.0
Resources:
  - MyFunction:
      Type: AWS::Lambda::Function
      Properties:
        Name: my-lambda-function
        Alias: live
        CurrentVersion: "1"
        TargetVersion: "2"

Hooks:
  - BeforeAllowTraffic: "PreTrafficHook"
  - AfterAllowTraffic: "PostTrafficHook"
```

### Deployment Strategies

**In-Place Deployment**
- Stop application on instances
- Deploy new version
- Start application
- Downtime during deployment
- Used for: EC2/On-Premises

**Blue/Green Deployment**
- Create new environment (Green)
- Deploy to Green environment
- Test Green environment
- Switch traffic to Green
- Keep Blue for rollback
- Zero downtime
- Used for: EC2, ECS, Lambda

**Deployment Configurations**
| Configuration | Description | Use Case |
|--------------|-------------|----------|
| **CodeDeployDefault.OneAtATime** | Deploy to one instance at a time | Maximum availability |
| **CodeDeployDefault.HalfAtATime** | Deploy to half of instances | Balance speed and availability |
| **CodeDeployDefault.AllAtOnce** | Deploy to all instances simultaneously | Fastest, but downtime |
| **CodeDeployDefault.LambdaCanary10Percent5Minutes** | 10% traffic for 5 min, then 100% | Lambda gradual rollout |
| **CodeDeployDefault.LambdaLinear10PercentEvery1Minute** | Increase 10% every minute | Lambda linear rollout |
| **CodeDeployDefault.ECSCanary10Percent5Minutes** | 10% traffic for 5 min, then 100% | ECS canary |
| **CodeDeployDefault.ECSLinear10PercentEvery1Minute** | Increase 10% every minute | ECS linear |

**Custom Deployment Configuration**
```yaml
Resources:
  CustomDeploymentConfig:
    Type: AWS::CodeDeploy::DeploymentConfig
    Properties:
      DeploymentConfigName: Custom25Percent
      MinimumHealthyHosts:
        Type: FLEET_PERCENT
        Value: 75  # 75% must remain healthy = deploy to 25% at a time
```

## Deployment Strategies Deep Dive

### Blue/Green Deployment

**Benefits**
- Zero downtime
- Easy rollback
- Full testing in production environment
- Reduced risk

**Challenges**
- Increased cost (duplicate environment)
- Database schema compatibility
- Complex for stateful applications

**ECS Blue/Green**
```yaml
Resources:
  ECSService:
    Type: AWS::ECS::Service
    Properties:
      Cluster: !Ref ECSCluster
      DeploymentController:
        Type: CODE_DEPLOY  # Enable blue/green
      LoadBalancers:
        - TargetGroupArn: !Ref BlueTargetGroup
          ContainerName: app
          ContainerPort: 80

  BlueTargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      Name: app-blue
      Port: 80
      Protocol: HTTP
      VpcId: !Ref VPC
      HealthCheckPath: /health

  GreenTargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      Name: app-green
      Port: 80
      Protocol: HTTP
      VpcId: !Ref VPC
      HealthCheckPath: /health

  Listener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      LoadBalancerArn: !Ref LoadBalancer
      Port: 80
      Protocol: HTTP
      DefaultActions:
        - Type: forward
          TargetGroupArn: !Ref BlueTargetGroup

  TestListener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      LoadBalancerArn: !Ref LoadBalancer
      Port: 8080
      Protocol: HTTP
      DefaultActions:
        - Type: forward
          TargetGroupArn: !Ref GreenTargetGroup
```

### Canary Deployment

**Strategy**
- Route small percentage of traffic to new version
- Monitor metrics
- Gradually increase traffic
- Rollback if issues detected

**Lambda Canary with Alias**
```yaml
Resources:
  LambdaFunction:
    Type: AWS::Lambda::Function
    Properties:
      FunctionName: my-function
      Code:
        S3Bucket: !Ref DeploymentBucket
        S3Key: function.zip

  LambdaVersion:
    Type: AWS::Lambda::Version
    Properties:
      FunctionName: !Ref LambdaFunction

  LambdaAlias:
    Type: AWS::Lambda::Alias
    Properties:
      FunctionName: !Ref LambdaFunction
      FunctionVersion: !GetAtt LambdaVersion.Version
      Name: live
      RoutingConfig:
        AdditionalVersionWeights:
          - FunctionVersion: !GetAtt NewLambdaVersion.Version
            FunctionWeight: 0.1  # 10% canary traffic
```

**API Gateway Canary**
```yaml
Resources:
  Deployment:
    Type: AWS::ApiGateway::Deployment
    Properties:
      RestApiId: !Ref RestApi
      Description: Production deployment
      DeploymentCanarySettings:
        PercentTraffic: 10.0
        StageVariableOverrides:
          lambdaAlias: canary
        UseStageCache: false

  Stage:
    Type: AWS::ApiGateway::Stage
    Properties:
      RestApiId: !Ref RestApi
      DeploymentId: !Ref Deployment
      StageName: prod
      CanarySettings:
        PercentTraffic: 10.0
        DeploymentId: !Ref Deployment
```

### Rolling Deployment

**Strategy**
- Update instances in batches
- Maintain minimum healthy instances
- Gradual rollout reduces risk
- Some instances may run old version during deployment

**Auto Scaling Rolling Update**
```yaml
Resources:
  LaunchTemplate:
    Type: AWS::EC2::LaunchTemplate
    Properties:
      LaunchTemplateName: app-template
      LaunchTemplateData:
        ImageId: !Ref LatestAMI
        InstanceType: t3.medium
        UserData:
          Fn::Base64: !Sub |
            #!/bin/bash
            # Install application

  AutoScalingGroup:
    Type: AWS::AutoScaling::AutoScalingGroup
    UpdatePolicy:
      AutoScalingRollingUpdate:
        MinInstancesInService: 2
        MaxBatchSize: 2
        PauseTime: PT5M
        WaitOnResourceSignals: true
        SuspendProcesses:
          - HealthCheck
          - ReplaceUnhealthy
          - AZRebalance
          - AlarmNotification
          - ScheduledActions
    Properties:
      LaunchTemplate:
        LaunchTemplateId: !Ref LaunchTemplate
        Version: !GetAtt LaunchTemplate.LatestVersionNumber
      MinSize: 4
      MaxSize: 8
      DesiredCapacity: 4
      TargetGroupARNs:
        - !Ref TargetGroup
      VPCZoneIdentifier:
        - !Ref Subnet1
        - !Ref Subnet2
```

### Feature Toggles

**Implementation**
```python
import boto3
import json

ssm = boto3.client('ssm')
appconfig = boto3.client('appconfig')

# Using SSM Parameter Store
def is_feature_enabled(feature_name):
    try:
        parameter = ssm.get_parameter(
            Name=f'/app/features/{feature_name}',
            WithDecryption=True
        )
        return json.loads(parameter['Parameter']['Value'])['enabled']
    except:
        return False

# Using AWS AppConfig
def get_feature_config():
    response = appconfig.get_configuration(
        Application='my-app',
        Environment='production',
        Configuration='feature-flags',
        ClientId='unique-client-id'
    )
    return json.loads(response['Content'].read())

# Usage in application
if is_feature_enabled('new_checkout_flow'):
    # New feature code
    process_checkout_v2()
else:
    # Old feature code
    process_checkout_v1()
```

**AppConfig Configuration**
```yaml
Resources:
  AppConfigApplication:
    Type: AWS::AppConfig::Application
    Properties:
      Name: my-application

  AppConfigEnvironment:
    Type: AWS::AppConfig::Environment
    Properties:
      ApplicationId: !Ref AppConfigApplication
      Name: production

  AppConfigProfile:
    Type: AWS::AppConfig::ConfigurationProfile
    Properties:
      ApplicationId: !Ref AppConfigApplication
      Name: feature-flags
      LocationUri: hosted
      Type: AWS.AppConfig.FeatureFlags

  AppConfigDeploymentStrategy:
    Type: AWS::AppConfig::DeploymentStrategy
    Properties:
      Name: Canary10Percent20Minutes
      DeploymentDurationInMinutes: 20
      GrowthFactor: 10
      ReplicateTo: NONE
```

## GitFlow and Branching Strategies

### GitFlow Model

**Branch Types**
- `main`: Production-ready code
- `develop`: Integration branch
- `feature/*`: New features
- `release/*`: Release preparation
- `hotfix/*`: Production bug fixes

**Workflow**
```bash
# Start new feature
git checkout -b feature/new-feature develop
# Develop feature
git commit -am "Add new feature"
# Merge back to develop
git checkout develop
git merge --no-ff feature/new-feature
git branch -d feature/new-feature

# Create release
git checkout -b release/1.2.0 develop
# Fix release bugs
git commit -am "Fix release bugs"
# Merge to main and tag
git checkout main
git merge --no-ff release/1.2.0
git tag -a 1.2.0
# Merge back to develop
git checkout develop
git merge --no-ff release/1.2.0

# Hotfix
git checkout -b hotfix/1.2.1 main
git commit -am "Fix critical bug"
git checkout main
git merge --no-ff hotfix/1.2.1
git tag -a 1.2.1
git checkout develop
git merge --no-ff hotfix/1.2.1
```

### Trunk-Based Development

**Strategy**
- Single main branch
- Short-lived feature branches
- Frequent integration
- Feature toggles for incomplete features

**Pipeline for Trunk-Based**
```yaml
# Trigger on main branch
- Name: Source
  Actions:
    - Name: Source
      ActionTypeId:
        Category: Source
        Owner: AWS
        Provider: CodeCommit
        Version: '1'
      Configuration:
        RepositoryName: my-repo
        BranchName: main  # Only main branch

# Continuous deployment
- Name: Deploy
  Actions:
    - Name: DeployProduction
      ActionTypeId:
        Category: Deploy
        Owner: AWS
        Provider: CodeDeploy
        Version: '1'
      Configuration:
        ApplicationName: !Ref Application
        DeploymentGroupName: production
```

## Testing in CI/CD

### Test Pyramid

**Levels**
1. Unit Tests (70%) - Fast, isolated
2. Integration Tests (20%) - Component interaction
3. End-to-End Tests (10%) - Full system

**Unit Tests in CodeBuild**
```yaml
phases:
  build:
    commands:
      - npm test -- --coverage --watchAll=false
      - python -m pytest tests/ --cov=src --cov-report=xml

reports:
  unit_tests:
    files:
      - 'test-results.xml'
    file-format: 'JUNITXML'
  coverage:
    files:
      - 'coverage.xml'
    file-format: 'COBERTURAXML'
```

**Integration Tests**
```yaml
- Name: IntegrationTest
  Actions:
    - Name: DeployTestEnv
      ActionTypeId:
        Category: Deploy
        Owner: AWS
        Provider: CloudFormation
        Version: '1'
      Configuration:
        ActionMode: CREATE_UPDATE
        StackName: test-environment
        TemplatePath: BuildOutput::template.yaml
        ParameterOverrides: |
          {
            "Environment": "test"
          }

    - Name: RunIntegrationTests
      RunOrder: 2
      ActionTypeId:
        Category: Test
        Owner: AWS
        Provider: CodeBuild
        Version: '1'
      Configuration:
        ProjectName: !Ref IntegrationTestProject

    - Name: TearDownTestEnv
      RunOrder: 3
      ActionTypeId:
        Category: Deploy
        Owner: AWS
        Provider: CloudFormation
        Version: '1'
      Configuration:
        ActionMode: DELETE_ONLY
        StackName: test-environment
```

## Exam Tips

### Key Concepts
- CodePipeline: Orchestrates CI/CD workflow
- CodeBuild: Compiles, tests, produces artifacts
- CodeDeploy: Automates deployments
- Blue/Green: Zero downtime, easy rollback
- Canary: Gradual traffic shift
- Rolling: Batch updates maintaining capacity

### Common Scenarios
- **Fast feedback needed**: Parallel test actions
- **Zero downtime required**: Blue/green deployment
- **Gradual rollout**: Canary with CloudWatch alarms
- **Multi-region**: Cross-region actions in pipeline
- **Multi-account**: Cross-account IAM roles
- **Docker builds**: CodeBuild with privileged mode
- **Automated rollback**: CloudWatch alarms with CodeDeploy

### Best Practices
- Automate everything possible
- Test early and often
- Fast feedback loops
- Immutable artifacts
- Environment parity
- Security scanning in pipeline
- Automated rollback mechanisms
- Version everything
