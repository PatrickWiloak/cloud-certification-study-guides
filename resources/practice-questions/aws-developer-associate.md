# AWS Developer Associate (DVA-C02) Practice Questions

40 scenario-based practice questions covering all exam domains.

**Exam Domain Breakdown:**
- Domain 1: Development with AWS Services (32%) - 13 questions
- Domain 2: Security (26%) - 10 questions
- Domain 3: Deployment (24%) - 10 questions
- Domain 4: Troubleshooting and Optimization (18%) - 7 questions

---

## Domain 1: Development with AWS Services (32%)

### Question 1
**Scenario:** A developer is building a serverless API that needs to validate request payloads before processing. The validation logic is complex and varies by endpoint. What's the most efficient approach?

A. Add validation code at the beginning of each Lambda function
B. Use API Gateway request validation with JSON Schema
C. Create a Lambda authorizer for validation
D. Use API Gateway mapping templates to validate requests

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** API Gateway request validation with JSON Schema provides built-in validation before Lambda is invoked, reducing costs and latency. Custom Lambda code adds execution time and cost. Lambda authorizers are for authentication, not payload validation. Mapping templates transform data but aren't designed for complex validation.

**Key Concept:** [API Gateway Request Validation](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-method-request-validation.html)
</details>

---

### Question 2
**Scenario:** A Lambda function processes messages from an SQS queue. During testing, some messages fail and are processed multiple times before going to the dead-letter queue. How can the developer ensure each message is processed only once?

A. Enable FIFO queue
B. Implement idempotency using DynamoDB with conditional writes
C. Decrease the visibility timeout
D. Increase the batch size

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Implementing idempotency using DynamoDB with conditional writes ensures that even if a message is delivered multiple times, the processing logic only executes once. FIFO queues provide ordering but don't prevent reprocessing on failure. Visibility timeout changes don't solve the idempotency problem. Batch size doesn't affect duplicate processing.

**Key Concept:** [Lambda Idempotency](https://docs.aws.amazon.com/lambda/latest/dg/invocation-eventfiltering.html)
</details>

---

### Question 3
**Scenario:** A developer needs to store user session data that expires after 30 minutes of inactivity. The application has millions of users and requires sub-millisecond read latency. Which service should they use?

A. DynamoDB with TTL enabled
B. RDS MySQL with scheduled cleanup
C. ElastiCache Redis with key expiration
D. S3 with lifecycle policies

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** ElastiCache Redis provides sub-millisecond latency and native TTL support for automatic key expiration, making it ideal for session management. DynamoDB TTL can take up to 48 hours to delete expired items. RDS doesn't provide sub-millisecond latency. S3 is for object storage, not session data.

**Key Concept:** [ElastiCache for Redis](https://docs.aws.amazon.com/AmazonElastiCache/latest/red-ug/WhatIs.html)
</details>

---

### Question 4
**Scenario:** A developer is building a Lambda function that needs to make 100 parallel API calls to an external service. The function times out at the default 3-second limit. What should the developer do?

A. Increase the Lambda timeout to 15 minutes
B. Use Lambda Provisioned Concurrency
C. Implement async processing with Step Functions
D. Increase Lambda memory allocation

<details>
<summary>Answer</summary>

**Correct: D**

**Why:** Increasing Lambda memory also increases CPU allocation proportionally, which speeds up parallel processing. The default timeout is 3 seconds, so increasing it alone won't help if the code is CPU-bound. Provisioned Concurrency reduces cold starts, not execution time. Step Functions add complexity unnecessary for this use case.

**Key Concept:** [Lambda Memory and CPU](https://docs.aws.amazon.com/lambda/latest/dg/configuration-memory.html)
</details>

---

### Question 5
**Scenario:** A developer needs to query a DynamoDB table by a non-key attribute. The queries are infrequent but need to return results quickly. What's the most cost-effective approach?

A. Use a Scan operation with a filter expression
B. Create a Global Secondary Index (GSI)
C. Create a Local Secondary Index (LSI)
D. Migrate to Amazon RDS

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** A Global Secondary Index allows efficient queries on non-key attributes without scanning the entire table. Since queries are infrequent, the GSI cost is minimal. Scan operations are expensive and slow for large tables. LSI can only be created at table creation and requires the same partition key. Migrating to RDS is unnecessary overhead.

**Key Concept:** [DynamoDB Global Secondary Indexes](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GSI.html)
</details>

---

### Question 6
**Scenario:** A developer's Lambda function needs to process records from a Kinesis stream. The function occasionally fails, causing the entire batch to be reprocessed. How can the developer improve this?

A. Enable enhanced fan-out
B. Configure bisect batch on function error
C. Increase the shard count
D. Use Lambda Destinations

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Bisect batch on function error splits a failed batch in half and retries, isolating the problematic record. This prevents good records from being reprocessed repeatedly. Enhanced fan-out improves throughput, not error handling. Increasing shards doesn't help with batch failures. Lambda Destinations handle async invocation results, not stream processing errors.

**Key Concept:** [Lambda Event Source Mapping](https://docs.aws.amazon.com/lambda/latest/dg/with-kinesis.html)
</details>

---

### Question 7
**Scenario:** A developer is building a REST API with API Gateway and Lambda. They need to return a custom error response with specific headers when validation fails. What's the correct approach?

A. Throw an exception in Lambda with the error details
B. Use API Gateway Gateway Responses
C. Configure a Lambda alias with error handling
D. Use API Gateway request validators only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Gateway Responses allow customization of error responses including status codes, headers, and body format for various error types. Throwing exceptions returns a generic 502 error. Lambda aliases don't affect error responses. Request validators trigger errors but don't customize the response format.

**Key Concept:** [API Gateway Gateway Responses](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-gatewayResponse-definition.html)
</details>

---

### Question 8
**Scenario:** A developer needs to upload a 10GB file to S3 from a client application. The upload must be resumable if interrupted. What approach should they use?

A. Single PUT request with increased timeout
B. Multipart upload with presigned URLs
C. AWS Transfer Family
D. S3 Transfer Acceleration

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Multipart upload breaks large files into parts that can be uploaded independently and resumed if interrupted. Presigned URLs allow secure uploads from clients without AWS credentials. Single PUT is limited to 5GB and not resumable. Transfer Family is for SFTP/FTPS. Transfer Acceleration improves speed but doesn't enable resumable uploads.

**Key Concept:** [S3 Multipart Upload](https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html)
</details>

---

### Question 9
**Scenario:** A developer needs to invoke a Lambda function asynchronously and process the result later. They want to automatically retry failed invocations and capture failures for analysis. What should they configure?

A. SQS queue as event source
B. Lambda Destinations with on-failure configuration
C. CloudWatch Events rule
D. SNS topic trigger

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Lambda Destinations allow routing async invocation results to different targets based on success or failure. This enables automatic retry handling and failure capture without additional infrastructure. SQS is for queue-based processing, not async invocation results. CloudWatch Events is for scheduling. SNS triggers invoke Lambda, not capture results.

**Key Concept:** [Lambda Destinations](https://docs.aws.amazon.com/lambda/latest/dg/invocation-async.html#invocation-async-destinations)
</details>

---

### Question 10
**Scenario:** A developer is building a GraphQL API using AWS AppSync. They need to combine data from a DynamoDB table and an external REST API in a single query. What's the best approach?

A. Use a Lambda resolver to call both data sources
B. Create separate resolvers and use pipeline resolvers
C. Use DynamoDB streams to sync data
D. Implement client-side data aggregation

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Pipeline resolvers in AppSync allow chaining multiple resolvers to combine data from different sources in a single GraphQL query. A single Lambda resolver would work but adds latency and complexity. DynamoDB streams are for data replication, not query-time aggregation. Client-side aggregation defeats the purpose of GraphQL.

**Key Concept:** [AppSync Pipeline Resolvers](https://docs.aws.amazon.com/appsync/latest/devguide/pipeline-resolvers.html)
</details>

---

### Question 11
**Scenario:** A developer's Lambda function makes API calls to a third-party service that has rate limits. During peak traffic, the function is throttled. How can the developer smooth out the request rate?

A. Use Lambda reserved concurrency
B. Add an SQS queue with controlled concurrency
C. Implement exponential backoff in the Lambda code
D. Use Lambda Provisioned Concurrency

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** An SQS queue with controlled Lambda concurrency limits the rate of requests to the external service, smoothing out bursts. Reserved concurrency limits function invocations but doesn't queue requests. Exponential backoff handles retries but doesn't prevent initial throttling. Provisioned Concurrency reduces cold starts, not request rate.

**Key Concept:** [SQS as Lambda Event Source](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html)
</details>

---

### Question 12
**Scenario:** A developer needs to store application configuration that may change without redeploying the application. The configuration includes feature flags and API endpoints. What service should they use?

A. Environment variables
B. AWS AppConfig
C. S3 bucket with versioning
D. Secrets Manager

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** AWS AppConfig provides managed configuration with validation, gradual rollout, and instant updates without redeployment. Environment variables require redeployment. S3 requires custom polling logic. Secrets Manager is for sensitive data like credentials, not general configuration.

**Key Concept:** [AWS AppConfig](https://docs.aws.amazon.com/appconfig/latest/userguide/what-is-appconfig.html)
</details>

---

### Question 13
**Scenario:** A developer needs to implement a fan-out pattern where a single event triggers multiple independent Lambda functions. Order of execution doesn't matter. What's the simplest approach?

A. SNS topic with multiple Lambda subscribers
B. Step Functions parallel state
C. EventBridge with multiple rules
D. SQS queue with multiple consumers

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** SNS provides native fan-out where a single message triggers all subscribers simultaneously. It's simpler than Step Functions for independent parallel execution. EventBridge works but requires multiple rules. SQS delivers each message to only one consumer, not suitable for fan-out.

**Key Concept:** [SNS Fan-out Pattern](https://docs.aws.amazon.com/sns/latest/dg/sns-common-scenarios.html)
</details>

---

## Domain 2: Security (26%)

### Question 14
**Scenario:** A developer needs to allow a Lambda function to access a DynamoDB table in another AWS account. What's the most secure approach?

A. Use hardcoded access keys from the other account
B. Configure a cross-account IAM role and assume it
C. Make the DynamoDB table public
D. Use VPC peering between accounts

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cross-account IAM roles allow secure access without sharing credentials. The Lambda function assumes the role in the other account using STS. Hardcoded keys are a security anti-pattern. Public DynamoDB tables don't exist. VPC peering doesn't grant IAM permissions.

**Key Concept:** [Cross-Account Access](https://docs.aws.amazon.com/IAM/latest/UserGuide/tutorial_cross-account-with-roles.html)
</details>

---

### Question 15
**Scenario:** A developer needs to securely store database credentials that a Lambda function uses. The credentials must be rotated automatically every 30 days. Which service should they use?

A. Systems Manager Parameter Store SecureString
B. AWS Secrets Manager with rotation enabled
C. Environment variables with KMS encryption
D. S3 bucket with server-side encryption

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** AWS Secrets Manager provides automatic rotation for database credentials with built-in Lambda rotation functions. Parameter Store SecureString doesn't have built-in rotation. Environment variables require redeployment to rotate. S3 is not designed for secrets management.

**Key Concept:** [Secrets Manager Rotation](https://docs.aws.amazon.com/secretsmanager/latest/userguide/rotating-secrets.html)
</details>

---

### Question 16
**Scenario:** A developer is implementing authentication for an API Gateway REST API. The API needs to validate JWT tokens issued by a third-party identity provider. What's the recommended approach?

A. Lambda authorizer that validates the JWT
B. IAM authorization
C. API Gateway JWT authorizer (HTTP API)
D. Cognito User Pool authorizer

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** For REST APIs with third-party JWTs, a Lambda authorizer provides full control over token validation. HTTP API's built-in JWT authorizer doesn't work with REST APIs. IAM authorization is for AWS credentials. Cognito authorizers work with Cognito-issued tokens, not third-party tokens.

**Key Concept:** [Lambda Authorizers](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-use-lambda-authorizer.html)
</details>

---

### Question 17
**Scenario:** A developer needs to encrypt sensitive data before storing it in DynamoDB. The encryption key must be managed by AWS but dedicated to this application. What should they use?

A. DynamoDB default encryption
B. Customer managed key (CMK) in KMS
C. AWS managed key in KMS
D. Client-side encryption with a self-managed key

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** A customer managed key (CMK) in KMS is managed by AWS but dedicated to the application with full control over key policies. Default encryption uses AWS owned keys with no control. AWS managed keys are shared across services. Self-managed keys require more operational overhead.

**Key Concept:** [DynamoDB Encryption](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/EncryptionAtRest.html)
</details>

---

### Question 18
**Scenario:** A developer's Lambda function needs to access resources in a private VPC subnet. After configuring VPC access, the function can no longer reach the internet to call external APIs. What should the developer do?

A. Add an Internet Gateway to the VPC
B. Configure a NAT Gateway in a public subnet
C. Disable VPC access for the Lambda function
D. Add a public IP to the Lambda function

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Lambda functions in a VPC private subnet need a NAT Gateway to access the internet. The NAT Gateway should be in a public subnet with a route to an Internet Gateway. Internet Gateway alone doesn't help private subnets. Disabling VPC access loses private resource connectivity. Lambda functions can't have public IPs.

**Key Concept:** [Lambda VPC Networking](https://docs.aws.amazon.com/lambda/latest/dg/configuration-vpc.html)
</details>

---

### Question 19
**Scenario:** A developer wants to ensure that only their specific Lambda function can write to an S3 bucket. What's the most secure way to implement this?

A. S3 bucket policy with principal condition
B. IAM policy attached to Lambda execution role
C. S3 ACL with canonical user ID
D. Resource-based policy on Lambda

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** An S3 bucket policy with a principal condition restricting access to the specific Lambda execution role ARN ensures only that function can write. IAM policy grants the function access but doesn't prevent other principals from accessing the bucket. ACLs are legacy and less flexible. Resource-based policies on Lambda control who can invoke it, not what it can access.

**Key Concept:** [S3 Bucket Policies](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucket-policies.html)
</details>

---

### Question 20
**Scenario:** A developer needs to implement fine-grained access control for a DynamoDB table where users can only read their own records. The application uses Cognito for authentication. What should they use?

A. DynamoDB IAM policies with conditions
B. Cognito Identity Pool with IAM role mapping
C. Lambda authorizer with custom logic
D. DynamoDB streams with filtering

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cognito Identity Pool can map authenticated users to IAM roles with policies that use DynamoDB condition keys (like `dynamodb:LeadingKeys`) to restrict access to items matching the user's identity. This provides fine-grained access control at the IAM level. Lambda authorizers add latency. DynamoDB streams are for change data capture.

**Key Concept:** [Fine-Grained Access Control](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/specifying-conditions.html)
</details>

---

### Question 21
**Scenario:** A developer needs to call AWS services from an EC2 instance. The security team prohibits storing credentials on instances. What's the correct approach?

A. Use environment variables
B. Use an IAM instance profile
C. Store credentials in Secrets Manager
D. Use AWS CLI configure

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** IAM instance profiles allow EC2 instances to assume a role and get temporary credentials automatically. No credentials are stored on the instance. Environment variables would require storing credentials. Secrets Manager adds complexity for this use case. CLI configure stores credentials in files.

**Key Concept:** [IAM Roles for EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)
</details>

---

### Question 22
**Scenario:** A developer is building a public API that needs to be protected against common web exploits like SQL injection and cross-site scripting. What AWS service should they use?

A. AWS Shield
B. AWS WAF
C. Security Groups
D. Network ACLs

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** AWS WAF provides protection against web exploits including SQL injection and XSS through managed rules and custom rules. Shield protects against DDoS attacks. Security Groups and Network ACLs filter network traffic but don't inspect application-layer content.

**Key Concept:** [AWS WAF](https://docs.aws.amazon.com/waf/latest/developerguide/what-is-aws-waf.html)
</details>

---

### Question 23
**Scenario:** A developer needs to encrypt environment variables in a Lambda function. The application code needs to decrypt them at runtime. What should they do?

A. Enable Lambda environment variable encryption with KMS
B. Store encrypted values and decrypt in code
C. Use Secrets Manager instead
D. Enable KMS encryption on the Lambda deployment package

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Lambda automatically encrypts environment variables at rest using KMS. Enabling encryption helpers provides automatic decryption at runtime. Storing pre-encrypted values requires manual KMS calls. Secrets Manager is better for secrets that need rotation. Deployment package encryption doesn't help environment variables.

**Key Concept:** [Lambda Environment Variables](https://docs.aws.amazon.com/lambda/latest/dg/configuration-envvars.html)
</details>

---

## Domain 3: Deployment (24%)

### Question 24
**Scenario:** A developer wants to deploy a new version of a Lambda function with minimal risk. They want to gradually shift traffic to the new version while monitoring for errors. What should they use?

A. Lambda versions with weighted alias
B. Blue/green deployment with separate functions
C. Canary deployment with CodeDeploy
D. All-at-once deployment

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** CodeDeploy with Lambda supports canary deployments that shift a small percentage of traffic initially, then complete the shift if no errors occur. Weighted aliases require manual traffic shifting. Blue/green with separate functions adds operational overhead. All-at-once is risky.

**Key Concept:** [Lambda Deployment with CodeDeploy](https://docs.aws.amazon.com/codedeploy/latest/userguide/deployment-configurations.html#deployment-configurations-lambda)
</details>

---

### Question 25
**Scenario:** A developer needs to deploy a containerized application to ECS. The deployment should replace instances gradually with automatic rollback if health checks fail. What deployment type should they configure?

A. In-place deployment
B. Blue/green deployment
C. Rolling update
D. Canary deployment

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** ECS rolling updates replace tasks gradually while maintaining availability. If health checks fail, the deployment rolls back automatically. Blue/green creates a new task set (more complex). In-place isn't an ECS concept. Canary is for Lambda with CodeDeploy.

**Key Concept:** [ECS Deployment Types](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-types.html)
</details>

---

### Question 26
**Scenario:** A developer wants to use AWS SAM to define a serverless application. They need to configure environment-specific values like database endpoints. What's the best approach?

A. Hardcode values in template.yaml
B. Use SAM parameter overrides with different files per environment
C. Create separate template files per environment
D. Use Lambda environment variables only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** SAM supports parameter overrides using `--parameter-overrides` with different values per environment. This keeps a single template with environment-specific configurations. Hardcoding prevents reuse. Separate templates cause drift. Lambda environment variables don't handle all template configurations.

**Key Concept:** [SAM Parameter Overrides](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-deploy.html)
</details>

---

### Question 27
**Scenario:** A developer needs to automate the deployment pipeline for a Lambda function. The pipeline should run unit tests, build the code, and deploy to staging and production environments with approval between stages. What service should they use?

A. CodeBuild only
B. CodePipeline with CodeBuild and manual approval actions
C. Jenkins on EC2
D. GitHub Actions only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** CodePipeline orchestrates the full deployment workflow including CodeBuild for testing/building, manual approval actions for stage gates, and deployment actions. CodeBuild alone doesn't provide workflow orchestration. Jenkins requires self-management. GitHub Actions alone lacks native AWS deployment integration.

**Key Concept:** [CodePipeline](https://docs.aws.amazon.com/codepipeline/latest/userguide/welcome.html)
</details>

---

### Question 28
**Scenario:** A developer is using Elastic Beanstalk to deploy an application. They want to add custom configuration like installing additional packages and running scripts during deployment. What should they use?

A. .ebextensions configuration files
B. Custom AMI
C. User data scripts
D. CodeDeploy hooks

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** .ebextensions configuration files allow customizing Elastic Beanstalk environments including packages, files, commands, and services. Custom AMI requires ongoing maintenance. User data runs only at instance launch. CodeDeploy hooks aren't used with Elastic Beanstalk.

**Key Concept:** [Elastic Beanstalk Configuration](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/ebextensions.html)
</details>

---

### Question 29
**Scenario:** A developer needs to deploy infrastructure and application code together. The infrastructure includes a Lambda function, DynamoDB table, and API Gateway. What tool should they use?

A. CloudFormation only
B. AWS CDK
C. Terraform
D. AWS CLI scripts

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** AWS CDK allows defining infrastructure and application code in the same project using familiar programming languages. It synthesizes to CloudFormation. Plain CloudFormation requires YAML/JSON. Terraform is multi-cloud but less integrated. CLI scripts lack state management and idempotency.

**Key Concept:** [AWS CDK](https://docs.aws.amazon.com/cdk/v2/guide/home.html)
</details>

---

### Question 30
**Scenario:** A developer is deploying a containerized application. They want to use infrastructure as code but the team prefers YAML over programming languages. What should they use for container orchestration?

A. ECS with CloudFormation
B. EKS with Helm charts
C. AWS CDK with TypeScript
D. Custom Docker scripts

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** ECS with CloudFormation uses YAML templates to define container services declaratively. EKS with Helm also uses YAML but adds Kubernetes complexity. CDK uses programming languages. Custom scripts lack the benefits of infrastructure as code.

**Key Concept:** [ECS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/AWS_ECS.html)
</details>

---

### Question 31
**Scenario:** A developer needs to store and version Lambda function deployment packages. They want to maintain multiple versions and roll back if needed. What should they use?

A. S3 bucket with versioning
B. CodeCommit repository
C. Lambda layers
D. ECR repository

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** S3 with versioning stores deployment packages with version history, enabling rollbacks. CodeCommit stores source code, not built packages. Lambda layers are for shared dependencies. ECR is for container images, not Lambda ZIP packages.

**Key Concept:** [Lambda Deployment Package](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-package.html)
</details>

---

### Question 32
**Scenario:** A developer wants to test Lambda functions locally before deploying. They need to simulate API Gateway events and DynamoDB interactions. What tool should they use?

A. AWS SAM CLI with sam local invoke
B. LocalStack
C. Docker compose
D. Unit tests with mocks only

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** SAM CLI's sam local invoke and sam local start-api allow testing Lambda functions locally with simulated AWS events. It integrates with Docker for the Lambda runtime. LocalStack is third-party. Docker compose doesn't simulate AWS services. Mocks don't test actual Lambda behavior.

**Key Concept:** [SAM CLI Local Testing](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-local-invoke.html)
</details>

---

### Question 33
**Scenario:** A developer needs to share common code between multiple Lambda functions without including it in each deployment package. What's the recommended approach?

A. Create a shared Lambda layer
B. Use Lambda extensions
C. Deploy the code to S3 and download at runtime
D. Use a monorepo with shared modules

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Lambda layers allow packaging shared libraries and dependencies separately from function code. Functions can include up to 5 layers. Extensions are for monitoring/security tools. Downloading from S3 adds latency. Monorepos help with development but don't reduce package size.

**Key Concept:** [Lambda Layers](https://docs.aws.amazon.com/lambda/latest/dg/chapter-layers.html)
</details>

---

## Domain 4: Troubleshooting and Optimization (18%)

### Question 34
**Scenario:** A developer notices that their Lambda function has high latency during the first invocation after periods of inactivity. Subsequent invocations are fast. What's causing this and how can they fix it?

A. Network latency; use VPC endpoints
B. Cold start; use Provisioned Concurrency
C. Memory limits; increase memory allocation
D. Timeout settings; increase timeout

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cold starts occur when Lambda needs to initialize a new execution environment. Provisioned Concurrency keeps environments warm, eliminating cold starts. VPC endpoints don't affect cold starts. Memory affects execution speed, not initialization. Timeout doesn't reduce latency.

**Key Concept:** [Lambda Provisioned Concurrency](https://docs.aws.amazon.com/lambda/latest/dg/provisioned-concurrency.html)
</details>

---

### Question 35
**Scenario:** A developer's API Gateway returns 502 errors intermittently. The backend Lambda function logs show successful executions. What's the most likely cause?

A. Lambda function timeout
B. Response format not matching API Gateway integration
C. API Gateway throttling
D. VPC connectivity issues

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** API Gateway returns 502 when the Lambda response format doesn't match expected structure (missing statusCode, headers, or body). Successful Lambda logs confirm execution completed. Timeout would show in Lambda logs. Throttling returns 429. VPC issues would cause Lambda failures.

**Key Concept:** [API Gateway 502 Errors](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-known-issues.html)
</details>

---

### Question 36
**Scenario:** A developer needs to trace requests through an application with Lambda functions, API Gateway, and DynamoDB. They want to visualize the request flow and identify bottlenecks. What service should they use?

A. CloudWatch Logs Insights
B. AWS X-Ray
C. CloudWatch Metrics
D. AWS Config

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** X-Ray provides distributed tracing across AWS services, showing request flow, latency breakdown, and bottlenecks in a service map. CloudWatch Logs shows log data. CloudWatch Metrics shows aggregate data. AWS Config tracks configuration changes.

**Key Concept:** [AWS X-Ray](https://docs.aws.amazon.com/xray/latest/devguide/aws-xray.html)
</details>

---

### Question 37
**Scenario:** A developer's DynamoDB table is experiencing throttling during peak hours. The table uses on-demand capacity mode. What should they investigate first?

A. Partition key design causing hot partitions
B. Insufficient read/write capacity units
C. Global Secondary Index throttling
D. DynamoDB stream configuration

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Even with on-demand capacity, DynamoDB can throttle hot partitions if a single partition receives disproportionate traffic. Poor partition key design (like using a date) causes this. On-demand mode auto-scales capacity units. GSI throttling is separate but also related to hot keys. Streams don't cause read/write throttling.

**Key Concept:** [DynamoDB Partitions](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/bp-partition-key-design.html)
</details>

---

### Question 38
**Scenario:** A developer needs to analyze Lambda function errors and group them by error type. They want to set up alerts for new error patterns. What should they use?

A. CloudWatch Logs with metric filters
B. CloudWatch Contributor Insights
C. Lambda Insights
D. CloudWatch Application Insights

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Lambda Insights provides automatic collection of function metrics including errors grouped by type, plus custom dashboard for analysis. Metric filters require manual setup for each error pattern. Contributor Insights is for high-cardinality data. Application Insights is for application health, not Lambda-specific errors.

**Key Concept:** [Lambda Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Lambda-Insights.html)
</details>

---

### Question 39
**Scenario:** A developer's Lambda function is running out of memory. They see "Process exited before completing request" in the logs. The function processes images from S3. What should they do?

A. Increase Lambda timeout
B. Increase Lambda memory
C. Use S3 Transfer Acceleration
D. Enable Lambda SnapStart

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** "Process exited before completing request" typically indicates the function ran out of memory. Image processing is memory-intensive. Increasing memory also increases CPU proportionally. Timeout doesn't help memory issues. Transfer Acceleration speeds downloads. SnapStart reduces cold starts.

**Key Concept:** [Lambda Memory Configuration](https://docs.aws.amazon.com/lambda/latest/dg/configuration-memory.html)
</details>

---

### Question 40
**Scenario:** A developer notices their SQS-triggered Lambda function is processing messages slowly. The function takes 10 seconds per message, and there are 1000 messages in the queue. How can they improve throughput?

A. Increase batch size
B. Increase Lambda reserved concurrency
C. Use FIFO queue
D. Decrease visibility timeout

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Increasing Lambda concurrency allows more function instances to process messages in parallel. With 10 seconds per message, higher concurrency directly improves throughput. Batch size processes multiple messages per invocation but each still takes 10 seconds. FIFO queues limit throughput. Decreasing visibility timeout risks duplicate processing.

**Key Concept:** [SQS Lambda Concurrency](https://docs.aws.amazon.com/lambda/latest/dg/with-sqs.html)
</details>

---

## Answer Key

| Question | Answer | Domain |
|----------|--------|--------|
| 1 | B | Development |
| 2 | B | Development |
| 3 | C | Development |
| 4 | D | Development |
| 5 | B | Development |
| 6 | B | Development |
| 7 | B | Development |
| 8 | B | Development |
| 9 | B | Development |
| 10 | B | Development |
| 11 | B | Development |
| 12 | B | Development |
| 13 | A | Development |
| 14 | B | Security |
| 15 | B | Security |
| 16 | A | Security |
| 17 | B | Security |
| 18 | B | Security |
| 19 | A | Security |
| 20 | B | Security |
| 21 | B | Security |
| 22 | B | Security |
| 23 | A | Security |
| 24 | C | Deployment |
| 25 | C | Deployment |
| 26 | B | Deployment |
| 27 | B | Deployment |
| 28 | A | Deployment |
| 29 | B | Deployment |
| 30 | A | Deployment |
| 31 | A | Deployment |
| 32 | A | Deployment |
| 33 | A | Deployment |
| 34 | B | Troubleshooting |
| 35 | B | Troubleshooting |
| 36 | B | Troubleshooting |
| 37 | A | Troubleshooting |
| 38 | C | Troubleshooting |
| 39 | B | Troubleshooting |
| 40 | B | Troubleshooting |

---

## Study Resources

- [AWS Developer Associate Exam Guide](https://aws.amazon.com/certification/certified-developer-associate/)
- [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/)
- [API Gateway Developer Guide](https://docs.aws.amazon.com/apigateway/latest/developerguide/)
- [DynamoDB Developer Guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/)
- [AWS SAM Developer Guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/)
