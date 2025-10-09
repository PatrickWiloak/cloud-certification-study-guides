# Amazon API Gateway

## Service Overview and Purpose

Amazon API Gateway is a fully managed service that makes it easy for developers to create, publish, maintain, monitor, and secure APIs at any scale. API Gateway handles all the tasks involved in accepting and processing up to hundreds of thousands of concurrent API calls.

### API Types
- **REST APIs**: RESTful APIs with resource-based operations
- **HTTP APIs**: Lower cost, higher performance alternative to REST APIs
- **WebSocket APIs**: Real-time two-way communication applications

## Key Features and Capabilities

### Core Features
- **Request/Response Transformation**: Modify requests and responses
- **Authentication and Authorization**: Multiple auth mechanisms
- **Rate Limiting**: Throttling and quotas
- **Caching**: Response caching for performance
- **Monitoring**: CloudWatch metrics and logging
- **CORS**: Cross-origin resource sharing support

### Integration Types
- **Lambda Function**: Serverless backend integration
- **HTTP**: Integration with HTTP endpoints
- **AWS Service**: Direct integration with AWS services
- **Mock**: Return static responses
- **VPC Link**: Private integration with VPC resources

## Configuration Examples

### REST API Creation
```bash
# Create REST API
aws apigateway create-rest-api \
  --name "Production API" \
  --description "Main production API" \
  --endpoint-configuration types=REGIONAL \
  --tags Environment=Production,Team=Backend

# Get root resource ID
aws apigateway get-resources --rest-api-id api123 --query 'items[?path==`/`].id' --output text

# Create resource
aws apigateway create-resource \
  --rest-api-id api123 \
  --parent-id root123 \
  --path-part users

# Create method
aws apigateway put-method \
  --rest-api-id api123 \
  --resource-id resource456 \
  --http-method GET \
  --authorization-type AWS_IAM \
  --api-key-required

# Create Lambda integration
aws apigateway put-integration \
  --rest-api-id api123 \
  --resource-id resource456 \
  --http-method GET \
  --type AWS_PROXY \
  --integration-http-method POST \
  --uri arn:aws:apigateway:us-west-2:lambda:path/2015-03-31/functions/arn:aws:lambda:us-west-2:account:function:getUserFunction/invocations

# Deploy API
aws apigateway create-deployment \
  --rest-api-id api123 \
  --stage-name prod \
  --stage-description "Production deployment"
```

### HTTP API Creation
```bash
# Create HTTP API (simpler, cheaper)
aws apigatewayv2 create-api \
  --name "HTTP API" \
  --protocol-type HTTP \
  --target arn:aws:lambda:us-west-2:account:function:myFunction \
  --tags Environment=Production

# Create stage
aws apigatewayv2 create-stage \
  --api-id httpapi123 \
  --stage-name prod \
  --auto-deploy \
  --throttle-settings BurstLimit=1000,RateLimit=500

# Create route
aws apigatewayv2 create-route \
  --api-id httpapi123 \
  --route-key "GET /users/{id}" \
  --target integrations/integration123
```

### WebSocket API
```bash
# Create WebSocket API
aws apigatewayv2 create-api \
  --name "Chat WebSocket API" \
  --protocol-type WEBSOCKET \
  --route-selection-expression '$request.body.action'

# Create routes
aws apigatewayv2 create-route \
  --api-id wsapi123 \
  --route-key '$connect' \
  --target integrations/integration123

aws apigatewayv2 create-route \
  --api-id wsapi123 \
  --route-key '$disconnect' \
  --target integrations/integration456

aws apigatewayv2 create-route \
  --api-id wsapi123 \
  --route-key 'sendMessage' \
  --target integrations/integration789
```

## Authentication and Authorization

### API Keys
```bash
# Create API key
aws apigateway create-api-key \
  --name "Mobile App Key" \
  --description "API key for mobile application" \
  --enabled

# Create usage plan
aws apigateway create-usage-plan \
  --name "Basic Plan" \
  --description "Basic usage plan" \
  --throttle BurstLimit=1000,RateLimit=500 \
  --quota Limit=10000,Period=MONTH,Offset=0

# Associate API key with usage plan
aws apigateway create-usage-plan-key \
  --usage-plan-id plan123 \
  --key-id key456 \
  --key-type API_KEY
```

### Lambda Authorizers
```python
# Lambda authorizer function
import json

def lambda_handler(event, context):
    token = event['authorizationToken']

    # Validate token (your logic here)
    if token == 'valid-token':
        effect = 'Allow'
    else:
        effect = 'Deny'

    # Build policy
    policy = {
        'principalId': 'user123',
        'policyDocument': {
            'Version': '2012-10-17',
            'Statement': [{
                'Action': 'execute-api:Invoke',
                'Effect': effect,
                'Resource': event['methodArn']
            }]
        },
        'context': {
            'userId': 'user123',
            'userRole': 'admin'
        }
    }

    return policy

# Create authorizer
aws apigateway create-authorizer \
  --rest-api-id api123 \
  --name "TokenAuthorizer" \
  --type TOKEN \
  --authorizer-uri arn:aws:apigateway:us-west-2:lambda:path/2015-03-31/functions/arn:aws:lambda:us-west-2:account:function:authorizerFunction/invocations \
  --authorizer-credentials arn:aws:iam::account:role/APIGatewayAuthorizerRole \
  --identity-source method.request.header.Authorization
```

### Cognito Integration
```bash
# Create Cognito authorizer
aws apigateway create-authorizer \
  --rest-api-id api123 \
  --name "CognitoAuthorizer" \
  --type COGNITO_USER_POOLS \
  --provider-arns arn:aws:cognito-idp:us-west-2:account:userpool/us-west-2_ABC123 \
  --identity-source method.request.header.Authorization

# Apply authorizer to method
aws apigateway update-method \
  --rest-api-id api123 \
  --resource-id resource456 \
  --http-method GET \
  --patch-ops op=replace,path=/authorizationType,value=COGNITO_USER_POOLS op=replace,path=/authorizerId,value=authorizer789
```

## Advanced Features

### Request/Response Transformation
```bash
# Request mapping template
aws apigateway put-integration \
  --rest-api-id api123 \
  --resource-id resource456 \
  --http-method POST \
  --type AWS \
  --integration-http-method POST \
  --uri arn:aws:apigateway:us-west-2:dynamodb:action/PutItem \
  --credentials arn:aws:iam::account:role/APIGatewayDynamoDBRole \
  --request-templates '{
    "application/json": "{
      \"TableName\": \"Users\",
      \"Item\": {
        \"id\": {\"S\": \"$context.requestId\"},
        \"name\": {\"S\": \"$input.path(\"$.name\")\"},
        \"email\": {\"S\": \"$input.path(\"$.email\")\"}
      }
    }"
  }'

# Response mapping template
aws apigateway put-method-response \
  --rest-api-id api123 \
  --resource-id resource456 \
  --http-method POST \
  --status-code 200 \
  --response-models application/json=Empty

aws apigateway put-integration-response \
  --rest-api-id api123 \
  --resource-id resource456 \
  --http-method POST \
  --status-code 200 \
  --response-templates '{
    "application/json": "{
      \"id\": \"$input.path(\"$.Attributes.id.S\")\",
      \"message\": \"User created successfully\"
    }"
  }'
```

### Caching
```bash
# Enable caching on stage
aws apigateway update-stage \
  --rest-api-id api123 \
  --stage-name prod \
  --patch-ops op=replace,path=/cacheClusterEnabled,value=true op=replace,path=/cacheClusterSize,value=0.5

# Method-level cache settings
aws apigateway update-method \
  --rest-api-id api123 \
  --resource-id resource456 \
  --http-method GET \
  --patch-ops op=replace,path=/caching/enabled,value=true op=replace,path=/caching/ttlInSeconds,value=300 op=replace,path=/caching/cacheKeyParameters,value=method.request.querystring.userId
```

### VPC Link for Private Integration
```bash
# Create VPC Link (for REST APIs)
aws apigateway create-vpc-link \
  --name "Private ALB Link" \
  --description "VPC Link to private ALB" \
  --target-arns arn:aws:elasticloadbalancing:us-west-2:account:loadbalancer/net/private-nlb/1234567890123456

# Create VPC Link for HTTP APIs
aws apigatewayv2 create-vpc-link \
  --name "HTTP API VPC Link" \
  --subnet-ids subnet-12345678 subnet-87654321 \
  --security-group-ids sg-12345678
```

## Use Cases and Scenarios

### Serverless API Backend
```python
# Lambda function for API backend
import json
import boto3
from decimal import Decimal

dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Users')

def lambda_handler(event, context):
    http_method = event['httpMethod']
    path = event['path']

    try:
        if http_method == 'GET' and path == '/users':
            # List users
            response = table.scan()
            users = response['Items']

            return {
                'statusCode': 200,
                'headers': {
                    'Content-Type': 'application/json',
                    'Access-Control-Allow-Origin': '*'
                },
                'body': json.dumps(users, default=decimal_default)
            }

        elif http_method == 'POST' and path == '/users':
            # Create user
            user_data = json.loads(event['body'])

            table.put_item(Item=user_data)

            return {
                'statusCode': 201,
                'headers': {
                    'Content-Type': 'application/json',
                    'Access-Control-Allow-Origin': '*'
                },
                'body': json.dumps({'message': 'User created successfully'})
            }

    except Exception as e:
        return {
            'statusCode': 500,
            'headers': {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*'
            },
            'body': json.dumps({'error': str(e)})
        }

def decimal_default(obj):
    if isinstance(obj, Decimal):
        return float(obj)
    raise TypeError
```

### WebSocket Chat Application
```python
# WebSocket Lambda functions
import json
import boto3

dynamodb = boto3.resource('dynamodb')
connections_table = dynamodb.Table('WebSocketConnections')
apigateway = boto3.client('apigatewaymanagementapi',
    endpoint_url='https://wsapi123.execute-api.us-west-2.amazonaws.com/prod')

def connect_handler(event, context):
    connection_id = event['requestContext']['connectionId']

    # Store connection
    connections_table.put_item(
        Item={'connectionId': connection_id}
    )

    return {'statusCode': 200}

def disconnect_handler(event, context):
    connection_id = event['requestContext']['connectionId']

    # Remove connection
    connections_table.delete_item(
        Key={'connectionId': connection_id}
    )

    return {'statusCode': 200}

def send_message_handler(event, context):
    connection_id = event['requestContext']['connectionId']
    message_data = json.loads(event['body'])

    # Get all connections
    response = connections_table.scan()
    connections = response['Items']

    # Broadcast message to all connections
    for connection in connections:
        try:
            apigateway.post_to_connection(
                ConnectionId=connection['connectionId'],
                Data=json.dumps(message_data)
            )
        except apigateway.exceptions.GoneException:
            # Remove stale connection
            connections_table.delete_item(
                Key={'connectionId': connection['connectionId']}
            )

    return {'statusCode': 200}
```

## Monitoring and Troubleshooting

### CloudWatch Metrics and Logs
```bash
# Enable detailed CloudWatch metrics
aws apigateway update-stage \
  --rest-api-id api123 \
  --stage-name prod \
  --patch-ops op=replace,path=/metricsEnabled,value=true op=replace,path=/loggingLevel,value=INFO op=replace,path=/dataTraceEnabled,value=true

# Create CloudWatch alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "API-Gateway-4xx-Errors" \
  --alarm-description "API Gateway 4xx errors" \
  --metric-name 4XXError \
  --namespace AWS/ApiGateway \
  --statistic Sum \
  --period 300 \
  --threshold 10 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=ApiName,Value="Production API" Name=Stage,Value=prod

# Query CloudWatch Logs Insights
aws logs start-query \
  --log-group-name "API-Gateway-Execution-Logs_api123/prod" \
  --start-time $(date -d "1 hour ago" +%s) \
  --end-time $(date +%s) \
  --query-string 'fields @timestamp, @message | filter @message like /ERROR/ | sort @timestamp desc'
```

### X-Ray Tracing
```bash
# Enable X-Ray tracing
aws apigateway update-stage \
  --rest-api-id api123 \
  --stage-name prod \
  --patch-ops op=replace,path=/tracingConfig/tracingEnabled,value=true

# Lambda function with X-Ray
import json
from aws_xray_sdk.core import xray_recorder

@xray_recorder.capture('lambda_handler')
def lambda_handler(event, context):
    # Your function logic with X-Ray tracing
    return {
        'statusCode': 200,
        'body': json.dumps('Hello from Lambda!')
    }
```

## Security Best Practices

### Resource Policies
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "execute-api:Invoke",
      "Resource": "arn:aws:execute-api:us-west-2:account:api123/*/*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": ["203.0.113.0/24", "198.51.100.0/24"]
        }
      }
    }
  ]
}
```

### WAF Integration
```bash
# Associate WAF with API Gateway
aws wafv2 associate-web-acl \
  --web-acl-arn arn:aws:wafv2:us-west-2:account:regional/webacl/name/id \
  --resource-arn arn:aws:apigateway:us-west-2::/restapis/api123/stages/prod
```

## Exam-Specific Tips

### Key Exam Topics
1. **API Types**: REST vs HTTP vs WebSocket APIs
2. **Integration Types**: Lambda proxy vs Lambda custom, HTTP, AWS service
3. **Authentication**: API keys, Lambda authorizers, Cognito
4. **Caching**: Stage-level and method-level caching
5. **Monitoring**: CloudWatch metrics, logs, X-Ray tracing

### Common Exam Scenarios
1. **Serverless API**: API Gateway + Lambda + DynamoDB
2. **Authentication**: Cognito User Pools with API Gateway
3. **Rate Limiting**: Throttling and usage plans
4. **CORS**: Cross-origin resource sharing configuration
5. **Private APIs**: VPC endpoints and resource policies

### Exam Tips
- **Know the differences** between API types and when to use each
- **Understand integration patterns** and their use cases
- **Remember authentication methods** and their implementation
- **Know caching behavior** and TTL settings
- **Understand error handling** and response codes