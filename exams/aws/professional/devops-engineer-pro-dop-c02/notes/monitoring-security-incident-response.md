# Monitoring, Security, and Incident Response

## CloudWatch Monitoring

### Core Metrics and Dimensions

**EC2 Metrics**
- CPU Utilization, Network In/Out, Disk Read/Write
- Status Check Failed (System, Instance)
- Default: 5-minute intervals (1-minute with detailed monitoring)
- Custom metrics via CloudWatch Agent

**RDS Metrics**
- DatabaseConnections, FreeableMemory, FreeStorageSpace
- ReadIOPS, WriteIOPS, ReadLatency, WriteLatency
- CPUUtilization, SwapUsage
- Enhanced Monitoring for OS-level metrics

**ELB/ALB Metrics**
- RequestCount, TargetResponseTime
- HealthyHostCount, UnHealthyHostCount
- HTTPCode_Target_4XX_Count, HTTPCode_Target_5XX_Count
- ActiveConnectionCount, TargetConnectionErrorCount

**Lambda Metrics**
- Invocations, Duration, Errors, Throttles
- ConcurrentExecutions, DeadLetterErrors
- IteratorAge (for stream-based invocations)

### CloudWatch Agent Configuration

```json
{
  "agent": {
    "metrics_collection_interval": 60,
    "region": "us-east-1",
    "logfile": "/opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log"
  },
  "metrics": {
    "namespace": "CustomApp",
    "metrics_collected": {
      "cpu": {
        "measurement": [
          {
            "name": "cpu_usage_idle",
            "rename": "CPU_IDLE",
            "unit": "Percent"
          },
          "cpu_usage_iowait"
        ],
        "metrics_collection_interval": 60,
        "totalcpu": false
      },
      "disk": {
        "measurement": [
          {
            "name": "used_percent",
            "rename": "DISK_USED",
            "unit": "Percent"
          }
        ],
        "metrics_collection_interval": 60,
        "resources": [
          "*"
        ]
      },
      "mem": {
        "measurement": [
          {
            "name": "mem_used_percent",
            "rename": "MEM_USED",
            "unit": "Percent"
          }
        ],
        "metrics_collection_interval": 60
      },
      "statsd": {
        "service_address": ":8125",
        "metrics_collection_interval": 10,
        "metrics_aggregation_interval": 60
      }
    }
  },
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/app/application.log",
            "log_group_name": "/aws/ec2/application",
            "log_stream_name": "{instance_id}",
            "retention_in_days": 7,
            "timezone": "UTC"
          },
          {
            "file_path": "/var/log/nginx/error.log",
            "log_group_name": "/aws/ec2/nginx",
            "log_stream_name": "{instance_id}/error"
          }
        ]
      },
      "windows_events": {
        "collect_list": [
          {
            "event_name": "System",
            "event_levels": ["ERROR", "CRITICAL"],
            "log_group_name": "windows-system-events",
            "log_stream_name": "{instance_id}"
          }
        ]
      }
    }
  }
}
```

### Custom Metrics

```python
import boto3
from datetime import datetime

cloudwatch = boto3.client('cloudwatch')

# Put single metric
cloudwatch.put_metric_data(
    Namespace='MyApp',
    MetricData=[
        {
            'MetricName': 'OrdersProcessed',
            'Value': 42,
            'Unit': 'Count',
            'Timestamp': datetime.utcnow(),
            'Dimensions': [
                {
                    'Name': 'Environment',
                    'Value': 'Production'
                },
                {
                    'Name': 'Region',
                    'Value': 'us-east-1'
                }
            ]
        }
    ]
)

# Put multiple metrics
cloudwatch.put_metric_data(
    Namespace='MyApp',
    MetricData=[
        {
            'MetricName': 'ProcessingTime',
            'Value': 125.5,
            'Unit': 'Milliseconds',
            'StorageResolution': 1  # High-resolution (1-second)
        },
        {
            'MetricName': 'ErrorRate',
            'Value': 0.02,
            'Unit': 'Percent'
        }
    ]
)

# Metric with statistics
cloudwatch.put_metric_data(
    Namespace='MyApp',
    MetricData=[
        {
            'MetricName': 'RequestLatency',
            'StatisticValues': {
                'SampleCount': 1000,
                'Sum': 50000,
                'Minimum': 10,
                'Maximum': 500
            },
            'Unit': 'Milliseconds'
        }
    ]
)
```

### CloudWatch Alarms

**Comprehensive Alarm Configuration**
```yaml
Resources:
  HighCPUAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: HighCPUUtilization
      AlarmDescription: Alert when CPU exceeds 80%
      MetricName: CPUUtilization
      Namespace: AWS/EC2
      Statistic: Average
      Period: 300
      EvaluationPeriods: 2
      Threshold: 80
      ComparisonOperator: GreaterThanThreshold
      Dimensions:
        - Name: InstanceId
          Value: !Ref Instance
      AlarmActions:
        - !Ref SNSTopic
        - !Ref ScalingPolicy
      OKActions:
        - !Ref SNSTopic
      TreatMissingData: breaching

  CompositeAlarm:
    Type: AWS::CloudWatch::CompositeAlarm
    Properties:
      AlarmName: HighErrorRateAndHighLatency
      AlarmDescription: Triggers when both error rate and latency are high
      AlarmRule: !Sub |
        (ALARM(${HighErrorRateAlarm}) AND ALARM(${HighLatencyAlarm}))
      ActionsEnabled: true
      AlarmActions:
        - !Ref CriticalAlertTopic

  AnomalyDetectorAlarm:
    Type: AWS::CloudWatch::Alarm
    Properties:
      AlarmName: AnomalousRequestCount
      ComparisonOperator: LessThanLowerOrGreaterThanUpperThreshold
      EvaluationPeriods: 2
      Metrics:
        - Id: m1
          ReturnData: true
          MetricStat:
            Metric:
              Namespace: AWS/ApplicationELB
              MetricName: RequestCount
              Dimensions:
                - Name: LoadBalancer
                  Value: !GetAtt LoadBalancer.LoadBalancerFullName
            Period: 300
            Stat: Sum
        - Id: ad1
          Expression: ANOMALY_DETECTION_BAND(m1, 2)
      ThresholdMetricId: ad1
      AlarmActions:
        - !Ref SNSTopic
```

### CloudWatch Logs Insights

**Query Examples**
```
# Find errors in last hour
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 100

# Count errors by type
fields @message
| filter @message like /ERROR/
| parse @message /ERROR: (?<errorType>.*?) -/
| stats count() by errorType
| sort count desc

# Analyze API latency
fields @timestamp, duration, statusCode, path
| filter statusCode >= 200
| stats avg(duration), max(duration), min(duration), count() by path
| sort avg(duration) desc

# Find slow queries
fields @timestamp, query, duration
| filter duration > 1000
| sort duration desc
| limit 20

# Track user activity
fields @timestamp, userId, action
| filter action in ["login", "logout", "purchase"]
| stats count() by userId, action

# Detect anomalies
fields @timestamp, requestCount
| stats avg(requestCount) as avg_requests, stddev(requestCount) as stddev_requests
| filter requestCount > (avg_requests + 2 * stddev_requests)
```

### CloudWatch Dashboards

```python
import json
import boto3

cloudwatch = boto3.client('cloudwatch')

dashboard_body = {
    "widgets": [
        {
            "type": "metric",
            "properties": {
                "metrics": [
                    ["AWS/EC2", "CPUUtilization", {"stat": "Average"}],
                    [".", ".", {"stat": "Maximum"}]
                ],
                "period": 300,
                "stat": "Average",
                "region": "us-east-1",
                "title": "EC2 CPU Utilization",
                "yAxis": {
                    "left": {"min": 0, "max": 100}
                }
            }
        },
        {
            "type": "log",
            "properties": {
                "query": """
                    SOURCE '/aws/lambda/my-function'
                    | fields @timestamp, @message
                    | filter @message like /ERROR/
                    | sort @timestamp desc
                    | limit 50
                """,
                "region": "us-east-1",
                "title": "Recent Errors"
            }
        },
        {
            "type": "metric",
            "properties": {
                "metrics": [
                    ["AWS/ApplicationELB", "TargetResponseTime",
                     {"stat": "Average", "label": "Avg Response Time"}],
                    ["...", {"stat": "p99", "label": "P99 Response Time"}]
                ],
                "view": "timeSeries",
                "stacked": false,
                "region": "us-east-1",
                "title": "ALB Response Times"
            }
        }
    ]
}

cloudwatch.put_dashboard(
    DashboardName='Production-Overview',
    DashboardBody=json.dumps(dashboard_body)
)
```

## AWS X-Ray

### Distributed Tracing

**X-Ray SDK Integration**
```python
# Python Flask application
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware
from flask import Flask

app = Flask(__name__)

# Configure X-Ray
xray_recorder.configure(
    service='my-api',
    context_missing='LOG_ERROR',
    plugins=('EC2Plugin', 'ECSPlugin'),
    sampling_rules='sampling-rules.json'
)

XRayMiddleware(app, xray_recorder)

@app.route('/api/orders')
@xray_recorder.capture('process_order')
def process_order():
    # Custom subsegment
    subsegment = xray_recorder.begin_subsegment('database_query')
    try:
        result = database.query('SELECT * FROM orders')
        subsegment.put_metadata('row_count', len(result))
        subsegment.put_annotation('query_type', 'select')
        return result
    except Exception as e:
        subsegment.add_exception(e)
        raise
    finally:
        xray_recorder.end_subsegment()

# Async operations
@xray_recorder.capture_async('async_operation')
async def fetch_user_data(user_id):
    async with aiohttp.ClientSession() as session:
        async with session.get(f'https://api.example.com/users/{user_id}') as response:
            return await response.json()
```

**Sampling Rules**
```json
{
  "version": 2,
  "rules": [
    {
      "description": "High priority endpoints",
      "host": "*",
      "http_method": "*",
      "url_path": "/api/critical/*",
      "fixed_target": 10,
      "rate": 1.0
    },
    {
      "description": "Health checks - low sampling",
      "host": "*",
      "http_method": "GET",
      "url_path": "/health",
      "fixed_target": 0,
      "rate": 0.01
    },
    {
      "description": "Default rule",
      "host": "*",
      "http_method": "*",
      "url_path": "*",
      "fixed_target": 1,
      "rate": 0.05
    }
  ],
  "default": {
    "fixed_target": 1,
    "rate": 0.1
  }
}
```

## EventBridge (CloudWatch Events)

### Event Patterns and Rules

```yaml
Resources:
  EC2StateChangeRule:
    Type: AWS::Events::Rule
    Properties:
      Name: EC2StateChangeNotification
      Description: Notify when EC2 instances stop
      EventPattern:
        source:
          - aws.ec2
        detail-type:
          - EC2 Instance State-change Notification
        detail:
          state:
            - stopped
            - terminated
      State: ENABLED
      Targets:
        - Arn: !GetAtt NotificationLambda.Arn
          Id: NotifyLambda
        - Arn: !Ref SNSTopic
          Id: SNSTarget

  CodePipelineFailureRule:
    Type: AWS::Events::Rule
    Properties:
      EventPattern:
        source:
          - aws.codepipeline
        detail-type:
          - CodePipeline Pipeline Execution State Change
        detail:
          state:
            - FAILED
          pipeline:
            - !Ref Pipeline
      Targets:
        - Arn: !GetAtt IncidentResponseLambda.Arn
          Id: IncidentResponse

  ScheduledRule:
    Type: AWS::Events::Rule
    Properties:
      ScheduleExpression: 'cron(0 2 * * ? *)'  # 2 AM daily
      State: ENABLED
      Targets:
        - Arn: !GetAtt BackupLambda.Arn
          Id: DailyBackup
          Input: |
            {
              "backupType": "full",
              "retention": 30
            }

  CustomEventRule:
    Type: AWS::Events::Rule
    Properties:
      EventPattern:
        source:
          - custom.myapp
        detail-type:
          - Order Placed
        detail:
          orderValue:
            - numeric:
              - ">"
              - 1000
      Targets:
        - Arn: !Ref PriorityQueue
          Id: HighValueOrders
```

## Security Best Practices

### IAM Roles and Policies

**Least Privilege Policy Example**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowEC2ReadOnly",
      "Effect": "Allow",
      "Action": [
        "ec2:Describe*",
        "ec2:Get*",
        "ec2:List*"
      ],
      "Resource": "*"
    },
    {
      "Sid": "AllowSpecificInstanceActions",
      "Effect": "Allow",
      "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances",
        "ec2:RebootInstances"
      ],
      "Resource": "arn:aws:ec2:*:*:instance/*",
      "Condition": {
        "StringEquals": {
          "ec2:ResourceTag/Environment": "Development"
        }
      }
    },
    {
      "Sid": "DenyInstanceTermination",
      "Effect": "Deny",
      "Action": "ec2:TerminateInstances",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "ec2:ResourceTag/Protected": "true"
        }
      }
    }
  ]
}
```

**Cross-Account Access**
```json
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

### Secrets Management

**AWS Secrets Manager**
```python
import boto3
import json
from botocore.exceptions import ClientError

secrets = boto3.client('secretsmanager')

def get_secret(secret_name):
    """Retrieve secret from Secrets Manager with caching"""
    try:
        response = secrets.get_secret_value(SecretId=secret_name)

        if 'SecretString' in response:
            return json.loads(response['SecretString'])
        else:
            return base64.b64decode(response['SecretBinary'])
    except ClientError as e:
        if e.response['Error']['Code'] == 'ResourceNotFoundException':
            print(f"Secret {secret_name} not found")
        raise e

def rotate_secret(secret_name):
    """Trigger secret rotation"""
    response = secrets.rotate_secret(
        SecretId=secret_name,
        RotationLambdaARN='arn:aws:lambda:region:account:function:rotate-secret',
        RotationRules={
            'AutomaticallyAfterDays': 30
        }
    )
    return response

# Usage
db_credentials = get_secret('prod/database/credentials')
database_url = f"postgresql://{db_credentials['username']}:{db_credentials['password']}@{db_credentials['host']}:5432/mydb"
```

**Parameter Store**
```python
import boto3

ssm = boto3.client('ssm')

# Store parameter
ssm.put_parameter(
    Name='/app/prod/db_password',
    Value='secure-password',
    Type='SecureString',
    KeyId='alias/aws/ssm',
    Tier='Standard',
    Tags=[
        {'Key': 'Environment', 'Value': 'Production'},
        {'Key': 'Application', 'Value': 'MyApp'}
    ]
)

# Get parameter
response = ssm.get_parameter(
    Name='/app/prod/db_password',
    WithDecryption=True
)
password = response['Parameter']['Value']

# Get parameters by path
response = ssm.get_parameters_by_path(
    Path='/app/prod/',
    Recursive=True,
    WithDecryption=True
)

config = {param['Name'].split('/')[-1]: param['Value']
          for param in response['Parameters']}
```

### Encryption

**S3 Bucket Encryption**
```yaml
Resources:
  EncryptedBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: my-encrypted-bucket
      BucketEncryption:
        ServerSideEncryptionConfiguration:
          - ServerSideEncryptionByDefault:
              SSEAlgorithm: aws:kms
              KMSMasterKeyID: !GetAtt KMSKey.Arn
            BucketKeyEnabled: true
      PublicAccessBlockConfiguration:
        BlockPublicAcls: true
        BlockPublicPolicy: true
        IgnorePublicAcls: true
        RestrictPublicBuckets: true

  KMSKey:
    Type: AWS::KMS::Key
    Properties:
      Description: S3 encryption key
      KeyPolicy:
        Version: '2012-10-17'
        Statement:
          - Sid: Enable IAM policies
            Effect: Allow
            Principal:
              AWS: !Sub 'arn:aws:iam::${AWS::AccountId}:root'
            Action: 'kms:*'
            Resource: '*'
          - Sid: Allow S3 to use key
            Effect: Allow
            Principal:
              Service: s3.amazonaws.com
            Action:
              - 'kms:Decrypt'
              - 'kms:GenerateDataKey'
            Resource: '*'
```

## Incident Response

### Automated Incident Response

**GuardDuty Finding Response**
```python
import boto3
import json

ec2 = boto3.client('ec2')
sns = boto3.client('sns')

def lambda_handler(event, context):
    """Respond to GuardDuty findings"""

    finding = event['detail']
    severity = finding['severity']
    finding_type = finding['type']

    if 'recon' in finding_type.lower():
        # Port scanning detected
        instance_id = finding['resource']['instanceDetails']['instanceId']

        # Isolate instance
        isolate_instance(instance_id)

        # Create snapshot for forensics
        create_forensic_snapshot(instance_id)

        # Notify security team
        notify_security_team(finding)

    elif 'trojan' in finding_type.lower() or 'backdoor' in finding_type.lower():
        # Malware detected
        instance_id = finding['resource']['instanceDetails']['instanceId']

        # Immediate isolation
        isolate_instance(instance_id)

        # Revoke IAM credentials
        instance_profile = get_instance_profile(instance_id)
        if instance_profile:
            disable_iam_credentials(instance_profile)

        # Escalate to critical
        escalate_incident(finding, 'CRITICAL')

def isolate_instance(instance_id):
    """Apply restrictive security group"""
    # Create forensics security group
    forensic_sg = ec2.create_security_group(
        GroupName=f'forensic-isolation-{instance_id}',
        Description='Isolation for incident response',
        VpcId=get_instance_vpc(instance_id)
    )

    # No inbound, only outbound to logging
    ec2.authorize_security_group_egress(
        GroupId=forensic_sg['GroupId'],
        IpPermissions=[
            {
                'IpProtocol': 'tcp',
                'FromPort': 443,
                'ToPort': 443,
                'IpRanges': [{'CidrIp': '0.0.0.0/0'}]
            }
        ]
    )

    # Apply to instance
    ec2.modify_instance_attribute(
        InstanceId=instance_id,
        Groups=[forensic_sg['GroupId']]
    )

def create_forensic_snapshot(instance_id):
    """Create EBS snapshots for forensic analysis"""
    volumes = ec2.describe_volumes(
        Filters=[{'Name': 'attachment.instance-id', 'Values': [instance_id]}]
    )

    for volume in volumes['Volumes']:
        ec2.create_snapshot(
            VolumeId=volume['VolumeId'],
            Description=f'Forensic snapshot - GuardDuty finding - {instance_id}',
            TagSpecifications=[
                {
                    'ResourceType': 'snapshot',
                    'Tags': [
                        {'Key': 'Forensic', 'Value': 'true'},
                        {'Key': 'InstanceId', 'Value': instance_id}
                    ]
                }
            ]
        )

def notify_security_team(finding):
    """Send detailed notification"""
    message = {
        'severity': finding['severity'],
        'type': finding['type'],
        'description': finding['description'],
        'resource': finding['resource'],
        'time': finding['time']
    }

    sns.publish(
        TopicArn='arn:aws:sns:region:account:security-incidents',
        Subject=f"GuardDuty Finding: {finding['type']}",
        Message=json.dumps(message, indent=2)
    )
```

### Runbook Automation with Systems Manager

```yaml
Resources:
  IncidentResponseDocument:
    Type: AWS::SSM::Document
    Properties:
      DocumentType: Automation
      Content:
        schemaVersion: '0.3'
        description: Automated incident response for compromised instance
        parameters:
          InstanceId:
            type: String
            description: ID of the compromised instance
          IsolationSecurityGroupId:
            type: String
            description: Security group for isolation
        mainSteps:
          - name: CreateForensicSnapshot
            action: 'aws:executeAwsApi'
            inputs:
              Service: ec2
              Api: CreateSnapshot
              VolumeId: '{{ InstanceId }}'
              Description: 'Forensic snapshot for incident response'
            outputs:
              - Name: SnapshotId
                Selector: $.SnapshotId
                Type: String

          - name: IsolateInstance
            action: 'aws:executeAwsApi'
            inputs:
              Service: ec2
              Api: ModifyInstanceAttribute
              InstanceId: '{{ InstanceId }}'
              Groups:
                - '{{ IsolationSecurityGroupId }}'

          - name: TagInstance
            action: 'aws:createTags'
            inputs:
              ResourceType: EC2
              ResourceIds:
                - '{{ InstanceId }}'
              Tags:
                - Key: Status
                  Value: Quarantined
                - Key: IncidentDate
                  Value: '{{ global:DATE_TIME }}'

          - name: NotifySecurityTeam
            action: 'aws:executeAwsApi'
            inputs:
              Service: sns
              Api: Publish
              TopicArn: 'arn:aws:sns:region:account:security-team'
              Subject: 'Incident Response Executed'
              Message: 'Instance {{ InstanceId }} has been isolated'
```

## Disaster Recovery

### Backup Strategies

**AWS Backup**
```yaml
Resources:
  BackupVault:
    Type: AWS::Backup::BackupVault
    Properties:
      BackupVaultName: production-backup-vault
      EncryptionKeyArn: !GetAtt BackupKMSKey.Arn

  BackupPlan:
    Type: AWS::Backup::BackupPlan
    Properties:
      BackupPlan:
        BackupPlanName: DailyBackupPlan
        BackupPlanRule:
          - RuleName: DailyBackup
            TargetBackupVault: !Ref BackupVault
            ScheduleExpression: 'cron(0 5 ? * * *)'
            StartWindowMinutes: 60
            CompletionWindowMinutes: 120
            Lifecycle:
              DeleteAfterDays: 30
              MoveToColdStorageAfterDays: 7
            RecoveryPointTags:
              Environment: Production
              BackupType: Daily

          - RuleName: WeeklyBackup
            TargetBackupVault: !Ref BackupVault
            ScheduleExpression: 'cron(0 5 ? * 1 *)'
            Lifecycle:
              DeleteAfterDays: 90
              MoveToColdStorageAfterDays: 30

  BackupSelection:
    Type: AWS::Backup::BackupSelection
    Properties:
      BackupPlanId: !Ref BackupPlan
      BackupSelection:
        SelectionName: ProductionResources
        IamRoleArn: !GetAtt BackupRole.Arn
        Resources:
          - !Sub 'arn:aws:ec2:${AWS::Region}:${AWS::AccountId}:instance/*'
          - !Sub 'arn:aws:rds:${AWS::Region}:${AWS::AccountId}:db:*'
        Conditions:
          StringEquals:
            - ConditionKey: 'aws:ResourceTag/Backup'
              ConditionValue: 'true'
```

## Exam Tips

### Monitoring
- CloudWatch for metrics, logs, alarms
- X-Ray for distributed tracing
- Enhanced monitoring for deeper insights
- Custom metrics for application-specific data
- Log Insights for log analysis
- Anomaly detection for baseline monitoring

### Security
- IAM roles over access keys
- Least privilege principle
- Secrets Manager for credentials rotation
- KMS for encryption at rest
- VPC endpoints for private connectivity
- Security groups as stateful firewalls
- GuardDuty for threat detection

### Incident Response
- Automated remediation with Lambda
- EventBridge for event-driven response
- Systems Manager for runbook automation
- Snapshots for forensic analysis
- Isolation before investigation
- Document and tag incidents

### Best Practices
- Monitor everything
- Automate responses
- Encrypt sensitive data
- Regular security audits
- Test disaster recovery plans
- Implement least privilege
- Use managed services for security
