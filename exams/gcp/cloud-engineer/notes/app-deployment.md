# Application Deployment - Cloud Engineer

## App Engine

### Environments

**Standard Environment**:
- Sandboxed runtime
- Rapid scaling (milliseconds)
- Free tier available
- Language runtimes: Python, Java, Node.js, PHP, Ruby, Go
- Restrictions on system access and libraries

**Flexible Environment**:
- Docker containers
- Gradual scaling (minutes)
- More flexibility, fewer restrictions
- Custom runtimes supported
- SSH access for debugging
- No free tier

### Key Features
- Automatic scaling
- Traffic splitting
- Versioning
- Built-in security
- Integrated monitoring

### Versions and Traffic Splitting
- Deploy multiple versions
- Split traffic between versions (gradual rollout)
- Canary testing
- A/B testing

### Scaling
**Standard**:
- Automatic: Scale based on load
- Basic: Scale based on requests
- Manual: Fixed instances

**Flexible**:
- Automatic: Target CPU/throughput
- Manual: Fixed instances

### Configuration: app.yaml
```yaml
runtime: python39
instance_class: F2
automatic_scaling:
  min_instances: 1
  max_instances: 10
  target_cpu_utilization: 0.6
```

## Cloud Functions

### Overview
- Event-driven serverless functions
- Pay per invocation and duration
- Auto-scaling
- No server management

### Generations
**1st gen**: Original Cloud Functions
**2nd gen**: Built on Cloud Run, more features

### Trigger Types
**HTTP**: HTTP(S) requests
**Cloud Storage**: Object events
**Pub/Sub**: Messages
**Firestore**: Document changes
**Firebase**: Authentication, analytics, etc.

### Runtime Environments
- Node.js, Python, Go, Java, .NET, Ruby, PHP

### Configuration
- Memory: 128MB to 8GB (2nd gen: up to 32GB)
- Timeout: Max 9 minutes (2nd gen: 60 minutes)
- Concurrency: 1 (1st gen), up to 1000 (2nd gen)

## Cloud Endpoints

### Overview
- API management platform
- Works with App Engine, GKE, Compute Engine
- OpenAPI specification
- API key validation
- Authentication
- Monitoring and logging

### Features
- API versioning
- Rate limiting
- Quota management
- API analytics
- Developer portal

## Cloud Tasks

### Overview
- Asynchronous task execution
- Guaranteed execution with retry
- Rate controls
- Schedule tasks for future execution

### Use Cases
- Offload work from web requests
- Background job processing
- Batch operations
- Scheduled tasks

### Queues
- HTTP target tasks
- App Engine tasks
- Rate limits and retry configuration

## Deployment Strategies

### Blue-Green Deployment
- Two identical environments
- Switch traffic between environments
- Quick rollback
- App Engine traffic splitting

### Canary Deployment
- Gradual rollout to subset of users
- Monitor for issues
- Incremental traffic increase
- App Engine versions and traffic splitting

### Rolling Updates
- Gradual update of instances
- Maintain availability
- GKE rolling updates
- Managed instance group updates

## Configuration Management

### Environment Variables
- App Engine: env_variables in app.yaml
- Cloud Functions: --set-env-vars flag
- Cloud Run: --set-env-vars flag

### Secrets
- Secret Manager integration
- Mount as environment variables
- Access via API
- Automatic rotation support

## Exam Tips

### Service Selection
- **App Engine**: PaaS web applications
- **Cloud Functions**: Event-driven, short tasks
- **Cloud Run**: Containerized applications, serverless
- **GKE**: Complex containerized workloads
- **Compute Engine**: Full VM control

### Key Differences
- **App Engine Standard vs Flexible**: Standard = faster scaling, limited runtime; Flexible = more control, Docker
- **Functions vs Run**: Functions = event-driven, simpler; Run = containers, more flexible
- **App Engine vs Cloud Run**: App Engine = traditional PaaS; Run = container-first

### Deployment Commands
```bash
# Deploy App Engine
gcloud app deploy

# Deploy Cloud Function
gcloud functions deploy FUNCTION_NAME \
  --runtime python39 \
  --trigger-http \
  --allow-unauthenticated

# Deploy Cloud Run
gcloud run deploy SERVICE_NAME \
  --image gcr.io/PROJECT/IMAGE \
  --platform managed \
  --region us-central1

# Traffic splitting (App Engine)
gcloud app services set-traffic SERVICE \
  --splits v1=.9,v2=.1
```

### Best Practices
1. Use Cloud Run for new containerized apps
2. Use Cloud Functions for event-driven tasks
3. Implement health checks
4. Use secrets manager for sensitive data
5. Enable logging and monitoring
6. Implement proper scaling policies
7. Use traffic splitting for gradual rollouts
