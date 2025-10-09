# Operations and Monitoring - GCP Associate Cloud Engineer

## Overview

This document covers Google Cloud operations, monitoring, and logging services including Cloud Monitoring, Cloud Logging, Error Reporting, Cloud Trace, and operational best practices. Managing and monitoring cloud solutions is a key domain for the Associate Cloud Engineer certification.

## Key Topics

### 1. Cloud Monitoring
- Infrastructure and application monitoring
- Metrics, dashboards, and alerting
- Uptime checks and SLO monitoring
- Custom metrics and monitoring agents

### 2. Cloud Logging
- Centralized log management
- Log collection, search, and analysis
- Log exports and sinks
- Audit logs for security and compliance

### 3. Error Reporting
- Real-time error tracking and alerting
- Error grouping and analysis
- Integration with applications
- Stack trace capture and display

### 4. Cloud Trace
- Distributed tracing for applications
- Latency analysis and performance insights
- Request flow visualization
- Integration with applications and services

### 5. Cloud Profiler
- Continuous CPU and memory profiling
- Production environment profiling
- Performance optimization insights
- Multiple language support

## GCP Services Reference

### Cloud Monitoring
- **Metrics**: Time-series data about resources and applications
- **Dashboards**: Visual displays of metrics and charts
- **Alerting Policies**: Notifications based on metric conditions
- **Uptime Checks**: Monitor endpoint availability and latency
- **Groups**: Organize resources for monitoring
- **Service Monitoring**: SLI/SLO/SLA tracking
- **Monitoring Agent**: Collect system and application metrics

### Cloud Logging
- **Logs Explorer**: Search and filter log entries
- **Log Sinks**: Export logs to destinations (Cloud Storage, BigQuery, Pub/Sub)
- **Log Buckets**: Store and manage log data
- **Log-based Metrics**: Create metrics from log entries
- **Exclusion Filters**: Reduce log storage costs
- **Cloud Audit Logs**: Admin, Data Access, System Event logs
- **Logging Agent**: Collect application and system logs

### Error Reporting
- **Error Groups**: Aggregated similar errors
- **Error Details**: Stack traces and context
- **Notifications**: Alert on new or frequent errors
- **Resolution Tracking**: Mark errors as resolved
- **Integration**: Works with Cloud Logging automatically

### Cloud Trace
- **Traces**: Request flow through distributed systems
- **Spans**: Individual operations within a trace
- **Trace List**: Browse and search traces
- **Analysis Reports**: Latency insights and trends
- **Integration**: Automatic for App Engine, manual for other services

### Cloud Profiler
- **CPU Profiling**: Identify CPU-intensive code
- **Heap Profiling**: Memory allocation analysis
- **Flame Graphs**: Visual representation of profiling data
- **Comparison Views**: Compare profiles across time periods

## Best Practices

### Cloud Monitoring Best Practices
1. **Meaningful Dashboards**: Create role-specific dashboards for different teams
2. **Effective Alerting**: Set thresholds that balance sensitivity and alert fatigue
3. **Notification Channels**: Use multiple channels for critical alerts
4. **Resource Groups**: Organize resources logically for easier monitoring
5. **Custom Metrics**: Instrument applications with business-relevant metrics
6. **SLO Monitoring**: Define and track service level objectives
7. **Uptime Checks**: Monitor critical endpoints from multiple locations
8. **Budget Alerts**: Set up billing alerts to avoid unexpected costs

### Cloud Logging Best Practices
1. **Log Retention**: Configure appropriate retention periods based on compliance needs
2. **Log Sinks**: Export logs for long-term storage or analysis
3. **Exclusion Filters**: Filter out noisy or unnecessary logs to reduce costs
4. **Structured Logging**: Use JSON for easier parsing and querying
5. **Log-based Metrics**: Create metrics from logs for monitoring and alerting
6. **Audit Logs**: Enable and monitor Admin Activity and Data Access logs
7. **Query Optimization**: Use efficient queries in Logs Explorer
8. **Access Control**: Restrict log access using IAM permissions

### Error Reporting Best Practices
1. **Error Context**: Include relevant context in error messages
2. **Error Handling**: Implement proper exception handling in applications
3. **Notification Setup**: Configure alerts for new error types
4. **Regular Review**: Check Error Reporting dashboard regularly
5. **Resolution Tracking**: Mark errors as resolved to track fixes
6. **Integration**: Ensure proper error reporting library integration
7. **Filtering**: Use filters to focus on critical errors

### Cloud Trace Best Practices
1. **Sampling Rate**: Adjust based on traffic volume and analysis needs
2. **Custom Spans**: Add spans for important application operations
3. **Trace Analysis**: Regularly review traces to identify performance bottlenecks
4. **Integration**: Enable tracing in all components of distributed systems
5. **Labels**: Add meaningful labels to spans for better filtering
6. **Performance Goals**: Use trace data to validate performance targets
7. **Correlation**: Link traces with logs and metrics for full observability

### Operational Excellence Best Practices
1. **Incident Response**: Develop and document incident response procedures
2. **Runbooks**: Create runbooks for common operational tasks
3. **Automation**: Automate repetitive operational tasks
4. **Change Management**: Implement controlled change processes
5. **Capacity Planning**: Monitor trends and plan for growth
6. **Post-Mortems**: Conduct blameless post-mortems after incidents
7. **Documentation**: Keep operational documentation up-to-date
8. **Testing**: Regularly test backup, recovery, and failover procedures

## Common Scenarios

### Scenario 1: Application Performance Monitoring
**Requirement**: Monitor application performance and receive alerts
**Solution**:
- Install Cloud Monitoring agent on Compute Engine instances
- Create custom dashboards showing key application metrics
- Set up alerting policies for high CPU, memory, or error rates
- Use Cloud Trace to identify slow requests
- Implement Cloud Profiler for code-level insights

### Scenario 2: Log Analysis for Troubleshooting
**Requirement**: Troubleshoot application issues using logs
**Solution**:
- Use Logs Explorer to search and filter application logs
- Create log-based metrics for important events
- Set up log sinks to export logs to BigQuery for analysis
- Use exclusion filters to reduce noise
- Correlate logs with metrics using time-based analysis

### Scenario 3: Security Audit and Compliance
**Requirement**: Monitor and audit user actions for compliance
**Solution**:
- Enable Admin Activity and Data Access audit logs
- Create log sinks to export audit logs to Cloud Storage
- Set up log-based metrics for security events
- Configure alerts for suspicious activities
- Use BigQuery for periodic compliance reports

### Scenario 4: Multi-Service Application Monitoring
**Requirement**: Monitor microservices architecture end-to-end
**Solution**:
- Implement Cloud Trace across all services
- Create unified dashboard showing all service health
- Set up Error Reporting for each service
- Configure cross-service alerting policies
- Use Service Monitoring for SLO tracking

### Scenario 5: Cost Optimization Monitoring
**Requirement**: Monitor and optimize Google Cloud costs
**Solution**:
- Create dashboards showing resource utilization
- Set up budget alerts for cost thresholds
- Use log-based metrics to track resource usage
- Monitor idle or underutilized resources
- Implement automated scaling based on metrics

## Study Tips

### Hands-On Practice
1. **Create Dashboards**: Build custom dashboards with different chart types
2. **Set Up Alerts**: Configure alerting policies with different conditions
3. **Explore Logs**: Practice searching and filtering logs in Logs Explorer
4. **Export Logs**: Create log sinks to BigQuery and Cloud Storage
5. **Use Error Reporting**: Deploy an application and intentionally create errors

### Key Concepts to Master
1. **Metrics vs Logs**: Understanding the difference and when to use each
2. **Alerting Policies**: Conditions, notifications, and documentation
3. **Log Retention**: Default retention and custom retention policies
4. **Audit Logs**: Types of audit logs and their purposes
5. **Observability**: Metrics, logs, and traces working together

### Common Exam Topics
1. Creating and managing monitoring dashboards
2. Configuring alerting policies with appropriate thresholds
3. Using Logs Explorer to search and filter logs
4. Setting up log exports to different destinations
5. Understanding Cloud Audit Logs types and usage
6. Implementing uptime checks for services
7. Using Cloud Trace for performance analysis
8. Creating log-based metrics

### gcloud Command Examples

```bash
# Cloud Monitoring - Alerting Policies (requires JSON/YAML config)
gcloud alpha monitoring policies create --policy-from-file=policy.yaml
gcloud alpha monitoring policies list

# Cloud Monitoring - Dashboards
gcloud monitoring dashboards create --config-from-file=dashboard.json
gcloud monitoring dashboards list

# Cloud Logging - Read Logs
gcloud logging read "resource.type=gce_instance" --limit=10 --format=json
gcloud logging read "severity>=ERROR" --limit=50

# Cloud Logging - Log Sinks
gcloud logging sinks create my-sink storage.googleapis.com/my-bucket --log-filter='resource.type=gce_instance'
gcloud logging sinks list
gcloud logging sinks describe my-sink
gcloud logging sinks update my-sink --log-filter='severity>=WARNING'
gcloud logging sinks delete my-sink

# Cloud Logging - Log Metrics
gcloud logging metrics create my-metric --description="Count of errors" --log-filter='severity=ERROR'
gcloud logging metrics list
gcloud logging metrics describe my-metric

# Cloud Logging - List log types
gcloud logging logs list
gcloud logging logs list --filter="LOG_ID('cloudaudit.googleapis.com')"

# Cloud Monitoring - Uptime Checks
gcloud monitoring uptime create my-check --display-name="My Website Check" --hostname=example.com --protocol=HTTPS
gcloud monitoring uptime list

# Install Monitoring and Logging agents on Compute Engine
curl -sSO https://dl.google.com/cloudagents/add-google-cloud-ops-agent-repo.sh
sudo bash add-google-cloud-ops-agent-repo.sh --also-install

# View logs from a specific resource
gcloud logging read "resource.type=gce_instance AND resource.labels.instance_id=INSTANCE_ID" --limit=20

# Create log exclusion
gcloud logging exclusions create my-exclusion --log-filter='resource.type=gce_instance AND severity<=INFO'

# View audit logs
gcloud logging read "logName:cloudaudit.googleapis.com" --limit=10 --format=json
```

### Logs Explorer Query Examples

```
# Find all errors
severity=ERROR

# Find errors from specific service
resource.type="gce_instance"
severity>=ERROR

# Find logs from specific instance
resource.labels.instance_id="INSTANCE_ID"

# Find logs with specific text
textPayload=~"database connection failed"

# Find logs in time range
timestamp>="2024-01-01T00:00:00Z"
timestamp<="2024-01-31T23:59:59Z"

# Combined filters
resource.type="cloud_run_revision"
severity>=WARNING
timestamp>="2024-01-01T00:00:00Z"

# Find admin activity audit logs
logName:"cloudaudit.googleapis.com/activity"

# Find data access audit logs
logName:"cloudaudit.googleapis.com/data_access"
```

## Additional Resources

- [Cloud Monitoring Documentation](https://cloud.google.com/monitoring/docs)
- [Cloud Logging Documentation](https://cloud.google.com/logging/docs)
- [Error Reporting Documentation](https://cloud.google.com/error-reporting/docs)
- [Cloud Trace Documentation](https://cloud.google.com/trace/docs)
- [Cloud Profiler Documentation](https://cloud.google.com/profiler/docs)
- [Operations Suite Overview](https://cloud.google.com/products/operations)
- [SRE Books](https://sre.google/books/) - Google's SRE practices
