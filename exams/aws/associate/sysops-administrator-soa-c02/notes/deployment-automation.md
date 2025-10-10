# Deployment, Provisioning, and Automation (SOA-C02)

This domain covers 18% of the exam and focuses on provisioning cloud resources and automating manual or repeatable processes.

## AWS CloudFormation

### Template Basics

**Template Sections**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Template description

Metadata:
  # Template metadata

Parameters:
  # Input parameters

Mappings:
  # Static variables/lookups

Conditions:
  # Conditional resource creation

Transform:
  # SAM or macros

Resources:
  # AWS resources (REQUIRED)

Outputs:
  # Values to export or display
```

**Resource Section** (Required)
```yaml
Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref LatestAmiId
      InstanceType: !Ref InstanceType
      Tags:
        - Key: Name
          Value: !Sub ${AWS::StackName}-instance
```

### Intrinsic Functions

**Ref** - Reference parameters or resources
```yaml
!Ref MyParameter
!Ref MyEC2Instance  # Returns resource ID
```

**GetAtt** - Get attribute from resource
```yaml
!GetAtt MyEC2Instance.PrivateIp
!GetAtt MyLoadBalancer.DNSName
```

**Sub** - String substitution
```yaml
!Sub 'arn:aws:s3:::${BucketName}/*'
!Sub |
  #!/bin/bash
  echo Hello ${AWS::StackName}
```

**Join** - Concatenate strings
```yaml
!Join [':', [a, b, c]]  # Returns "a:b:c"
```

**Split** - Split string into list
```yaml
!Split ['|', 'a|b|c']  # Returns [a, b, c]
```

**Select** - Select item from list
```yaml
!Select [0, !GetAZs '']  # First AZ in region
```

**FindInMap** - Lookup value in mapping
```yaml
Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-0123456789
!FindInMap [RegionMap, !Ref 'AWS::Region', AMI]
```

**Conditional Functions**
```yaml
Conditions:
  IsProduction: !Equals [!Ref Environment, production]

Resources:
  MyResource:
    Type: AWS::EC2::Instance
    Condition: IsProduction  # Only create in production
```

**ImportValue** - Import from another stack
```yaml
!ImportValue OtherStack-VPCId
```

### Stack Operations

**Create Stack**
- Validates template
- Creates resources in order (handles dependencies)
- Rolls back on failure (default) or disable
- Stack events track progress
- Tags propagated to resources

**Update Stack**
- Change sets preview changes
- Rolling back to previous template possible
- Updates may require replacement
- Stack policy prevents accidental updates
- Drift detection identifies manual changes

**Delete Stack**
- Deletes all resources in stack
- DeletionPolicy prevents resource deletion
- Retain, Snapshot, or Delete (default)
- Failed deletes require manual cleanup

### Advanced Features

**Nested Stacks**
- Reusable stack templates
- Overcome 500 resource limit
- Manage common patterns (VPC, security groups)
- Parent stack manages child stacks

```yaml
Resources:
  VPCStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/bucket/vpc.yaml
      Parameters:
        CidrBlock: 10.0.0.0/16
```

**Stack Sets**
- Deploy stacks across multiple accounts/regions
- AWS Organizations integration
- Automatic deployments to new accounts
- Permission models: Self-managed or service-managed

**Use Cases**
- Multi-account security baseline
- Standardized VPC configuration
- Cross-region DR deployments

**Change Sets**
- Preview stack changes before execution
- Review resource modifications
- Identify replacement vs update
- Execute or delete change set

**Drift Detection**
- Identify manual configuration changes
- Stack-level or resource-level detection
- Status: IN_SYNC, MODIFIED, DELETED, NOT_CHECKED
- Does not detect property value drift for all properties

**Custom Resources**
- Extend CloudFormation with Lambda
- Implement Create, Update, Delete logic
- Custom provisioning logic
- Integration with third-party services
- Helper scripts for signaling success/failure

**Stack Policy**
- Prevent updates to critical resources
- JSON document defining allowed actions
- Cannot be removed once set (only updated)
- Temporary override during updates

```json
{
  "Statement": [{
    "Effect": "Deny",
    "Action": "Update:*",
    "Principal": "*",
    "Resource": "LogicalResourceId/ProductionDatabase"
  }]
}
```

### Best Practices

1. **Use Parameters**: Make templates reusable
2. **Validate Templates**: Use `aws cloudformation validate-template`
3. **Use Change Sets**: Preview changes before execution
4. **Tag Resources**: Enable cost allocation and organization
5. **Enable Termination Protection**: Prevent accidental deletion
6. **Use Drift Detection**: Identify configuration drift
7. **Modularize**: Use nested stacks for complex architectures
8. **Version Control**: Track template changes in Git
9. **Use Helper Scripts**: cfn-init, cfn-signal, cfn-hup
10. **Least Privilege**: IAM permissions for stack operations

## AWS Elastic Beanstalk

### Platform Overview

**Supported Platforms**
- Docker (single or multi-container)
- Go, Java, .NET, Node.js, PHP, Python, Ruby
- Tomcat, Passenger, Puma
- Custom platforms using Packer

**Components**
- Application: Logical collection of environments
- Application Version: Specific iteration of deployable code
- Environment: Running version of application
- Environment Tier: Web server or worker
- Environment Configuration: Settings for environment

### Environment Types

**Web Server Tier**
- Handles HTTP requests
- Elastic Load Balancer
- Auto Scaling group
- EC2 instances running application
- CloudWatch metrics

**Worker Tier**
- Background processing
- SQS queue for messages
- Auto Scaling based on queue depth
- No load balancer
- Daemon pulls messages from queue

### Deployment Strategies

**All at Once**
- Deploys to all instances simultaneously
- Fastest deployment
- Downtime during deployment
- Failed deployment requires redeploy
- Good for development environments

**Rolling**
- Deploys in batches
- Reduced capacity during deployment
- No additional cost
- Longer deployment time
- Failed deployment requires additional rolling update

**Configuration**
- Batch size: Number of instances per batch
- Batch type: Percentage or fixed number

**Rolling with Additional Batch**
- Launches new batch first
- Maintains full capacity
- Small additional cost (temporary instances)
- Longer deployment time
- Good for production

**Immutable**
- Launches new instances in new ASG
- Full capacity maintained
- Zero downtime
- Quick rollback (terminate new ASG)
- Highest cost (double capacity temporarily)
- Safest for production

**Traffic Splitting (Canary)**
- New version receives percentage of traffic
- Monitor new version performance
- Auto-rollback if CloudWatch alarms triggered
- Blue/green deployment style
- Good for testing in production

**Blue/Green Deployment**
- Create new environment (green)
- Test thoroughly
- Swap URLs (CNAME swap)
- Instant rollback by swapping back
- Zero downtime
- Full environment duplication cost

### Configuration

**Environment Configuration Sources**
1. Elastic Beanstalk console/CLI
2. Saved configurations
3. Configuration files (.ebextensions)
4. Direct updates to resources

**Precedence** (highest to lowest)
1. Direct changes to resources (not recommended)
2. Environment properties
3. Saved configurations
4. Configuration files (.ebextensions)
5. Default values

**.ebextensions**
- YAML or JSON files in `.ebextensions/` directory
- Customize environment resources
- Run commands during deployment
- Install packages, create files, modify services

```yaml
# .ebextensions/01-custom.config
packages:
  yum:
    git: []

commands:
  01-command:
    command: "echo 'Hello World' > /tmp/hello.txt"

files:
  "/etc/app/config.conf":
    mode: "000644"
    owner: root
    group: root
    content: |
      # App configuration
      setting=value
```

### Monitoring and Logging

**Health Monitoring**
- Enhanced health reporting
- Health status: OK, Warning, Degraded, Severe, Info
- Health checks: Instance, environment, application
- Custom health checks via application

**Metrics**
- Environment metrics in CloudWatch
- Instance metrics, request metrics
- ELB metrics
- Custom CloudWatch metrics

**Logs**
- Request logs
- Application logs
- Server logs (Nginx, Apache)
- Stream to CloudWatch Logs
- Bundle and download logs

### Lifecycle Policies

**Application Version Limit**
- Maximum 1000 application versions
- Lifecycle policy auto-deletes old versions
- Based on age or count
- Optional S3 deletion

**Configuration**
- Max versions to retain
- Max age (days)
- Delete source bundle from S3

### Scaling

**Auto Scaling Configuration**
- Min/max instances
- Scaling triggers (CloudWatch metrics)
- Breach duration
- Upper/lower thresholds
- Scaling cooldown

**Common Metrics**
- NetworkOut
- CPUUtilization
- RequestCount
- TargetResponseTime
- SQS queue depth (worker tier)

## AWS Lambda Deployments

### Lambda Versions and Aliases

**Versions**
- Immutable snapshots of function
- Published versions numbered (1, 2, 3...)
- $LATEST: Unpublished, mutable version
- Each version has unique ARN
- Can't change code of published version

**Aliases**
- Pointer to Lambda version(s)
- Mutable (can update to point to different version)
- Named (dev, test, prod)
- Blue/green deployments with traffic shifting
- Use aliases in event sources (not versions)

**Traffic Shifting**
- Weighted alias: Split traffic between versions
- Canary: 10% to new version, 90% to old
- Linear: Gradually increase traffic to new version
- All-at-once: Immediate cutover
- CloudWatch alarms for automatic rollback

### SAM (Serverless Application Model)

**SAM Template**
- Simplified CloudFormation syntax for serverless
- Transforms to full CloudFormation
- Supports Lambda, API Gateway, DynamoDB, Step Functions

**SAM Transform**
```yaml
Transform: AWS::Serverless-2016-10-31

Resources:
  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: index.handler
      Runtime: python3.9
      CodeUri: ./src
      Events:
        Api:
          Type: Api
          Properties:
            Path: /hello
            Method: get
```

**SAM CLI Commands**
- `sam init`: Initialize new project
- `sam build`: Build application
- `sam local`: Test locally
- `sam deploy`: Deploy to AWS
- `sam logs`: Fetch logs

### Lambda Deployment Methods

**Console/CLI Upload**
- Zip file upload (up to 50MB)
- Inline editor for small functions
- Quick updates for development

**S3 Upload**
- Larger packages (up to 250MB unzipped)
- Better for large dependencies
- Versioning with S3 versioning

**Container Images**
- Package as container image
- Up to 10GB image size
- Use Docker for dependencies
- ECR for image storage

**CloudFormation/SAM**
- Infrastructure as code
- Repeatable deployments
- Version control
- Best practice for production

## EC2 Image Builder

### Overview

**Purpose**
- Automate AMI creation and maintenance
- Automated testing and validation
- Patch management for images
- Multi-region distribution

**Components**
1. **Image Pipeline**: Automation workflow
2. **Image Recipe**: How to build and test image
3. **Infrastructure Configuration**: Where to build
4. **Distribution Settings**: Where to publish

### Image Pipelines

**Pipeline Configuration**
- Schedule: Cron expression or manual
- Image recipe or container recipe
- Infrastructure configuration
- Distribution settings
- Image testing enabled/disabled
- Enhanced metadata collection

**Build Process**
1. Launch build instance from base image
2. Apply build components (install software, configurations)
3. Run test components
4. Create AMI or container image
5. Distribute to regions/accounts
6. Terminate build instance

### Image Recipes

**Components**
- Base image (source AMI or container)
- Build components (installation, configuration)
- Test components (validation)
- Block device mappings
- Working directory

**Build Components**
- Document format (similar to SSM documents)
- Phases: build, validate, test
- User-defined or AWS-managed
- Reusable across recipes

**Example Component**
```yaml
name: InstallSoftware
description: Install required software
schemaVersion: 1.0
phases:
  - name: build
    steps:
      - name: InstallApache
        action: ExecuteBash
        inputs:
          commands:
            - 'yum install httpd -y'
            - 'systemctl enable httpd'
```

### Testing and Validation

**Test Components**
- AWS-managed tests (reachability, etc.)
- Custom tests
- Block deployment on failure
- Test reports in S3

**Distribution**
- Multi-region copying
- Cross-account sharing
- Launch permissions
- AMI tags propagation

## AWS Systems Manager Automation

### Automation Documents

**Document Types**
- AWS-managed: Pre-built runbooks
- Custom: User-created
- Shared: From other accounts/community

**Common AWS Automation Documents**
- AWS-UpdateLinuxAmi: Patch and update AMI
- AWS-CreateSnapshot: Create EBS snapshot
- AWS-RestartEC2Instance: Restart instances
- AWS-DeleteSnapshot: Delete old snapshots
- AWS-ConfigureS3BucketLogging: Enable S3 logging

### Automation Actions

**Common Actions**
- aws:executeAwsApi: Call any AWS API
- aws:runInstances: Launch EC2 instances
- aws:createImage: Create AMI
- aws:runCommand: Execute Run Command document
- aws:sleep: Wait for specified time
- aws:waitForAwsResourceProperty: Wait for resource state
- aws:branch: Conditional execution
- aws:executeScript: Run Python or PowerShell script

**Example Automation Document**
```yaml
schemaVersion: '0.3'
description: Create AMI and delete old snapshots
parameters:
  InstanceId:
    type: String
    description: EC2 instance to create AMI from

mainSteps:
  - name: createImage
    action: aws:createImage
    inputs:
      InstanceId: '{{ InstanceId }}'
      ImageName: 'Automated-AMI-{{ global:DATE_TIME }}'

  - name: waitForImage
    action: aws:waitForAwsResourceProperty
    inputs:
      Service: ec2
      Api: DescribeImages
      ImageIds:
        - '{{ createImage.ImageId }}'
      PropertySelector: '$.Images[0].State'
      DesiredValues:
        - available
```

### Execution

**Execution Modes**
- Simple: Single account, single region
- Rate control: Concurrency and error threshold
- Multi-account/region: AWS Organizations integration
- Scheduled: Cron or rate expression

**Execution Inputs**
- Static values
- Parameters from calling service
- SSM Parameter Store values
- Outputs from previous steps

**Error Handling**
- Retry on failure
- OnFailure actions (abort, continue, step)
- Rollback on failure
- Timeout configuration

## AWS Service Catalog

### Overview

**Purpose**
- Centralized catalog of approved IT services
- Self-service portal for end users
- Governance and compliance
- Standardized deployments

**Benefits**
- Consistent resource configuration
- Cost control
- Rapid provisioning
- Reduced admin overhead
- Compliance enforcement

### Components

**Products**
- CloudFormation template
- Product versions
- Launch constraints
- Tags and metadata

**Portfolios**
- Collection of products
- Access control (IAM principals)
- Sharing across accounts
- Tag-based access control

**Constraints**
- Launch: IAM role for provisioning
- Template: Parameter restrictions
- Notification: SNS on provisioning events
- StackSet: Multi-account/region deployment

### Product Configuration

**Launch Constraint**
- IAM role assumed during provisioning
- Allows users to provision without full IAM permissions
- Best practice for security

**Template Constraint**
- Restrict parameter values
- Enforce governance rules
- Prevent non-compliant configurations

**Example**
```json
{
  "Rules": {
    "InstanceType": {
      "Assertions": [
        {
          "Assert": {
            "Fn::Contains": [
              ["t2.micro", "t2.small", "t3.micro"],
              {"Ref": "InstanceType"}
            ]
          },
          "AssertDescription": "Only t2/t3 micro and small allowed"
        }
      ]
    }
  }
}
```

### Best Practices

1. **Version Control**: Products and templates
2. **Least Privilege**: Launch constraints with minimal IAM
3. **Tag Everything**: Enable cost tracking
4. **Documentation**: Clear product descriptions
5. **Approval Workflow**: For high-risk products
6. **Monitoring**: Track provisioned products
7. **Regular Updates**: Keep products current

## Best Practices for Deployment and Automation

### Infrastructure as Code

1. **Version Control**: All templates in Git
2. **Code Review**: Pull requests for changes
3. **Testing**: Validate in dev before production
4. **Parameterization**: Reusable across environments
5. **Documentation**: Clear README and inline comments

### Deployment Safety

1. **Change Sets**: Preview CloudFormation changes
2. **Blue/Green**: Zero-downtime deployments
3. **Canary**: Gradual rollout with monitoring
4. **Rollback Plan**: Automated or documented
5. **Testing**: Automated testing in pipeline

### Automation

1. **Idempotent**: Safe to run multiple times
2. **Error Handling**: Graceful failure recovery
3. **Logging**: Comprehensive execution logs
4. **Notifications**: Alert on failures
5. **Documentation**: Runbook for manual intervention

## Exam Tips

### Key Concepts to Remember

- CloudFormation: Infrastructure as code, declarative, track dependencies
- Nested Stacks: Modular templates, overcome resource limits
- Stack Sets: Multi-account/region deployments
- Drift Detection: Identify manual configuration changes
- Elastic Beanstalk: Platform as a Service, easy deployment
- Deployment Strategies: All at once < Rolling < Rolling with batch < Immutable
- Blue/Green: Zero downtime, instant rollback via CNAME swap
- Lambda Aliases: Traffic shifting for canary deployments
- SAM: Simplified serverless template syntax
- Image Builder: Automated AMI creation and distribution
- Systems Manager Automation: Runbooks for operational tasks
- Service Catalog: Governance for approved AWS resources

### Common Scenarios

**Scenario**: Deploy application with zero downtime
**Solution**: Elastic Beanstalk blue/green or immutable deployment

**Scenario**: Automate AMI updates monthly with patching
**Solution**: EC2 Image Builder pipeline with schedule, test components

**Scenario**: Standardized VPC deployment across multiple accounts
**Solution**: CloudFormation Stack Sets with AWS Organizations

**Scenario**: Self-service EC2 provisioning with governance
**Solution**: Service Catalog portfolio with launch constraints

**Scenario**: Lambda canary deployment with automatic rollback
**Solution**: Lambda alias with traffic shifting, CloudWatch alarms

### Troubleshooting

**Issue**: CloudFormation stack creation failed
- Check stack events for specific error
- Verify IAM permissions
- Validate template syntax
- Check service limits
- Review parameter values

**Issue**: Elastic Beanstalk deployment failed
- Check health status and events
- Review logs (tail or bundle)
- Verify .ebextensions syntax
- Check application code errors
- Review instance security group rules

**Issue**: Systems Manager automation fails
- Check execution details and error messages
- Verify IAM permissions for automation role
- Review step inputs and outputs
- Test API calls manually
- Check resource availability

**Issue**: Image Builder pipeline fails
- Review build logs
- Check build component syntax
- Verify base image availability
- Check infrastructure configuration
- Review IAM permissions
