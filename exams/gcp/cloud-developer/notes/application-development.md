# Application Development - GCP Professional Cloud Developer

## Overview

Cloud-native application development covering App Engine, Cloud Run, Cloud Functions, containers, APIs, and development best practices.

## Key Topics

1. **Application Platforms** - App Engine, Cloud Run, GKE, Cloud Functions
2. **Development Tools** - Cloud Build, Cloud Source Repositories, Artifact Registry
3. **Application Design** - Microservices, serverless, event-driven, API-first
4. **Testing and Debugging** - Unit testing, integration testing, Cloud Debugger, Cloud Profiler

## Application Platforms

### App Engine
**Standard Environment**:
- Automatic scaling (including to zero)
- Fast startup, no cold start for some runtimes
- Limited language runtimes
- Sandbox restrictions
- Free tier included

**Flexible Environment**:
- Custom runtimes (any language with Dockerfile)
- SSH access to instances
- Longer startup time
- More expensive
- Custom machine types

**Use Cases**: Web apps, mobile backends, APIs, simple microservices

### Cloud Run
**Features**:
- Fully managed serverless containers
- Automatic scaling (including to zero)
- Request-based invocation
- HTTP/HTTPS only
- Pay per use (CPU/memory/requests)

**Best Practices**:
- Build minimal container images
- Use multi-stage builds
- Configure concurrency appropriately
- Set memory and CPU limits
- Use Cloud SQL with connection pooling
- Implement health checks

### Cloud Functions
**Triggers**:
- HTTP triggers
- Cloud Pub/Sub
- Cloud Storage events
- Firestore events
- Firebase events

**Languages**: Node.js, Python, Go, Java, .NET, Ruby, PHP

**Best Practices**:
- Keep functions small and focused
- Use environment variables for config
- Minimize cold starts
- Implement idempotency
- Use Secret Manager for secrets
- Handle timeouts gracefully

### GKE for Developers
**Advantages**:
- Full Kubernetes features
- Microservices orchestration
- Service mesh (Istio)
- Complex deployments
- Portability

**Development Tools**:
- Skaffold for local development
- Cloud Code for IDE integration
- Config Connector for GCP resources
- Workload Identity for authentication

## Development Workflow

### CI/CD with Cloud Build
**Build Configuration** (cloudbuild.yaml):
```yaml
steps:
  # Run tests
  - name: 'gcr.io/cloud-builders/npm'
    args: ['test']

  # Build container
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/$PROJECT_ID/myapp:$COMMIT_SHA', '.']

  # Push to registry
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'gcr.io/$PROJECT_ID/myapp:$COMMIT_SHA']

  # Deploy to Cloud Run
  - name: 'gcr.io/cloud-builders/gcloud'
    args: ['run', 'deploy', 'myapp', '--image', 'gcr.io/$PROJECT_ID/myapp:$COMMIT_SHA', '--region', 'us-central1']

images:
  - 'gcr.io/$PROJECT_ID/myapp:$COMMIT_SHA'
```

**Triggers**:
- GitHub/GitLab integration
- Branch-based triggers
- Tag-based triggers
- Manual triggers
- Webhook triggers

### Container Development
**Dockerfile Best Practices**:
```dockerfile
# Multi-stage build
FROM node:16 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Production image
FROM node:16-slim
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
EXPOSE 8080
CMD ["node", "dist/server.js"]
```

**Optimization**:
- Use specific base image tags
- Multi-stage builds
- Minimize layers
- .dockerignore file
- Use slim/alpine variants
- Cache dependencies

## Application Design Patterns

### Microservices Architecture
**Principles**:
- Single responsibility per service
- Independent deployment
- Loose coupling
- API-based communication
- Decentralized data

**Communication**:
- Synchronous: HTTP/gRPC
- Asynchronous: Pub/Sub
- Service mesh: Istio
- API Gateway: Cloud Endpoints, Apigee

### Event-Driven Architecture
**Components**:
- Event producers
- Event channels (Pub/Sub)
- Event consumers (Cloud Functions, Cloud Run)
- Event storage (Firestore, Bigtable)

**Patterns**:
- Event sourcing
- CQRS (Command Query Responsibility Segregation)
- Saga pattern for distributed transactions
- Event replay for recovery

### Serverless Patterns
**Benefits**:
- No server management
- Automatic scaling
- Pay per use
- Fast development

**Challenges**:
- Cold starts
- Execution time limits
- Stateless design
- Vendor lock-in

**When to Use**:
- Variable or unpredictable load
- Event-driven workloads
- Rapid prototyping
- Cost optimization

## API Development

### RESTful APIs
**Best Practices**:
- Use HTTP methods correctly (GET, POST, PUT, DELETE, PATCH)
- Versioning in URL or header
- Proper status codes
- Pagination for large results
- Rate limiting
- Authentication (OAuth 2.0, API keys)
- Documentation (OpenAPI/Swagger)

### gRPC APIs
**Advantages**:
- Protocol Buffers (efficient serialization)
- HTTP/2 (multiplexing, server push)
- Strongly typed contracts
- Code generation
- Streaming support

**Use Cases**:
- Microservices communication
- High-performance requirements
- Polyglot environments
- Real-time streaming

### API Management
**Cloud Endpoints**:
- API gateway for App Engine, Cloud Run, GKE
- Authentication and API key validation
- Rate limiting and quotas
- Monitoring and logging
- OpenAPI specification support

**Apigee**:
- Full API management platform
- Advanced security policies
- Monetization
- Developer portal
- Analytics

## Testing and Debugging

### Testing Strategies
**Unit Testing**:
- Test individual functions/methods
- Mock external dependencies
- Fast execution
- High code coverage

**Integration Testing**:
- Test component interactions
- Use test databases
- Cloud Functions: functions-framework for local testing
- Cloud Run: Deploy to test environment

**End-to-End Testing**:
- Test complete user workflows
- Automated browser testing
- Load testing for performance

### Cloud Debugger
**Features**:
- Debug production code without stopping
- Set snapshot breakpoints
- Conditional breakpoints
- View variable values
- No performance impact

**Supported**: App Engine, GKE, Compute Engine

### Cloud Profiler
**Capabilities**:
- CPU profiling
- Heap profiling
- Continuous profiling
- Production environment
- Minimal overhead

**Analysis**:
- Flame graphs
- Call hierarchy
- Time spent per function
- Memory allocation patterns

### Cloud Trace
**Distributed Tracing**:
- Request flow visualization
- Latency analysis
- Identify bottlenecks
- Service dependency mapping

**Implementation**:
- Automatic for App Engine
- OpenTelemetry integration
- Custom spans for key operations

## Best Practices

### Development
1. Use environment variables for configuration
2. Implement proper error handling
3. Use structured logging
4. Design for statelessness
5. Implement health checks
6. Version APIs properly
7. Use dependency injection
8. Follow 12-factor app principles

### Security
1. Never commit secrets to version control
2. Use Secret Manager or environment variables
3. Implement authentication and authorization
4. Validate all inputs
5. Use HTTPS for all communications
6. Apply principle of least privilege
7. Regular security scanning
8. Keep dependencies updated

### Performance
1. Use caching strategically
2. Optimize database queries
3. Implement connection pooling
4. Use CDN for static assets
5. Compress responses
6. Lazy load resources
7. Use asynchronous operations
8. Monitor and optimize continuously

## Common Scenarios

**Scenario**: Scalable web application with microservices
**Solution**: GKE or Cloud Run for services, Cloud SQL for database, Memorystore for caching, Cloud Load Balancing, Cloud CDN

**Scenario**: Event-driven data processing
**Solution**: Cloud Storage trigger → Cloud Functions → Transform data → Pub/Sub → Cloud Run subscriber → BigQuery

**Scenario**: Mobile app backend
**Solution**: Cloud Run or Cloud Functions for API, Firebase Authentication, Firestore for database, Cloud Storage for media

**Scenario**: CI/CD pipeline
**Solution**: Cloud Source Repositories → Cloud Build (test, build, containerize) → Artifact Registry → Deploy to Cloud Run/GKE

## Study Tips

1. **Hands-on practice**: Build and deploy applications to each platform
2. **Platform differences**: Know when to use App Engine vs Cloud Run vs GKE
3. **CI/CD configuration**: Practice Cloud Build configurations
4. **Testing tools**: Use Cloud Debugger and Profiler
5. **Design patterns**: Understand microservices and serverless patterns
6. **API development**: RESTful and gRPC best practices
7. **Container optimization**: Multi-stage builds, image sizing

## Key Commands

```bash
# App Engine
gcloud app deploy app.yaml
gcloud app versions list
gcloud app services set-traffic default --splits=v1=0.5,v2=0.5

# Cloud Run
gcloud run deploy SERVICE --image=gcr.io/PROJECT/IMAGE --region=us-central1 --allow-unauthenticated
gcloud run services update SERVICE --set-env-vars="KEY=VALUE"

# Cloud Functions
gcloud functions deploy FUNCTION --runtime=python39 --trigger-http --entry-point=main
gcloud functions call FUNCTION --data='{"key":"value"}'

# Cloud Build
gcloud builds submit --config=cloudbuild.yaml
gcloud builds triggers create github --repo-name=REPO --branch-pattern="^main$" --build-config=cloudbuild.yaml

# Artifact Registry
gcloud artifacts repositories create REPO --repository-format=docker --location=us-central1
docker push us-central1-docker.pkg.dev/PROJECT/REPO/IMAGE:TAG
```

## Additional Resources

- [App Engine Documentation](https://cloud.google.com/appengine/docs)
- [Cloud Run Documentation](https://cloud.google.com/run/docs)
- [Cloud Functions Documentation](https://cloud.google.com/functions/docs)
- [Cloud Build Documentation](https://cloud.google.com/build/docs)
- [12-Factor App](https://12factor.net/)
