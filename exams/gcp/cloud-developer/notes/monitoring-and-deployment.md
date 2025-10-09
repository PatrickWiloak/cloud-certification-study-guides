# Monitoring and Deployment - GCP Professional Cloud Developer

## Overview

Application monitoring, performance management, deployment strategies, and traffic management for cloud-native applications.

## Application Monitoring

### Cloud Monitoring
**Custom Metrics**:
```python
from google.cloud import monitoring_v3

client = monitoring_v3.MetricServiceClient()
project_name = f"projects/{project_id}"

# Write custom metric
series = monitoring_v3.TimeSeries()
series.metric.type = "custom.googleapis.com/my_metric"
series.resource.type = "global"
point = series.points.add()
point.value.double_value = 42.0
point.interval.end_time.seconds = int(time.time())

client.create_time_series(name=project_name, time_series=[series])
```

### Cloud Logging
**Structured Logging**:
```python
import logging
from google.cloud import logging as cloud_logging

client = cloud_logging.Client()
client.setup_logging()

# Structured log entry
logging.info("User action", extra={
    "user_id": "123",
    "action": "purchase",
    "amount": 99.99
})
```

### Error Reporting
```python
from google.cloud import error_reporting

client = error_reporting.Client()

try:
    risky_operation()
except Exception as e:
    client.report_exception()
```

## Deployment Strategies

### Blue-Green Deployment
**App Engine**:
```bash
# Deploy new version without traffic
gcloud app deploy --no-promote --version=v2

# Test new version
curl https://v2-dot-myapp.appspot.com

# Switch traffic
gcloud app services set-traffic default --splits=v2=1.0

# Rollback if needed
gcloud app services set-traffic default --splits=v1=1.0
```

### Canary Deployment
```bash
# Gradual rollout
gcloud app services set-traffic default --splits=v1=0.9,v2=0.1
# Monitor metrics
gcloud app services set-traffic default --splits=v1=0.5,v2=0.5
# Full rollout
gcloud app services set-traffic default --splits=v2=1.0
```

### Rolling Deployment (GKE)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1
  template:
    spec:
      containers:
      - name: app
        image: gcr.io/project/app:v2
```

## Performance Optimization

### Cloud Profiler
```python
import googlecloudprofiler

googlecloudprofiler.start(
    service='myapp',
    service_version='1.0.0',
    project_id='my-project'
)
```

### Cloud Trace
```python
from google.cloud import trace_v2

tracer = trace_v2.TracesServiceClient()

# Create span
span = {
    'display_name': 'operation_name',
    'start_time': start_time,
    'end_time': end_time
}
```

## Traffic Management

### Load Balancing
**Cloud Run**:
```bash
# Split traffic between revisions
gcloud run services update-traffic myservice \
  --to-revisions=myservice-v1=50,myservice-v2=50
```

**GKE with Istio**:
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: myapp
spec:
  hosts:
  - myapp.example.com
  http:
  - match:
    - headers:
        cookie:
          exact: "version=beta"
    route:
    - destination:
        host: myapp-v2
        weight: 100
  - route:
    - destination:
        host: myapp-v1
        weight: 90
    - destination:
        host: myapp-v2
        weight: 10
```

## Best Practices

### Monitoring
1. Implement health check endpoints
2. Log structured data
3. Use custom metrics for business KPIs
4. Set up alerting policies
5. Monitor latency, errors, saturation
6. Implement distributed tracing
7. Use dashboards for visualization

### Deployment
1. Use immutable deployments
2. Automate deployment pipeline
3. Test in staging environment
4. Implement canary/blue-green
5. Have rollback procedures
6. Monitor post-deployment
7. Use feature flags

### Performance
1. Profile application regularly
2. Optimize hot paths
3. Implement caching
4. Use CDN for static assets
5. Optimize database queries
6. Monitor resource usage
7. Load test before deployment

## Common Scenarios

**Scenario**: Zero-downtime deployment
**Solution**: Blue-green deployment with health checks and gradual traffic shift

**Scenario**: A/B testing
**Solution**: Traffic splitting based on headers/cookies, monitor metrics for each version

**Scenario**: Performance debugging
**Solution**: Cloud Profiler for CPU/memory, Cloud Trace for latency, Cloud Logging for errors

## Study Tips

1. Practice deployment strategies on App Engine and Cloud Run
2. Implement custom metrics and dashboards
3. Use Cloud Profiler and Trace
4. Understand traffic management patterns
5. Know rollback procedures
6. Practice with Cloud Build CI/CD

## Additional Resources

- [Cloud Monitoring](https://cloud.google.com/monitoring/docs)
- [Deployment Best Practices](https://cloud.google.com/architecture/application-deployment-and-testing-strategies)
- [Cloud Profiler](https://cloud.google.com/profiler/docs)
