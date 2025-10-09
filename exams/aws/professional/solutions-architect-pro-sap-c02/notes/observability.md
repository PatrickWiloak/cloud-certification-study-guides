# Observability

## Overview

Enterprise observability on AWS encompasses monitoring, logging, tracing, and alerting across distributed systems. At the professional level, you must design comprehensive observability strategies that provide visibility into application performance, infrastructure health, security events, and business metrics while enabling rapid troubleshooting, capacity planning, and operational insights.

Professional architects must implement centralized logging, distributed tracing for microservices, custom metrics for business KPIs, automated alerting with intelligent thresholds, and dashboards that provide actionable insights across multi-account, multi-region environments.

## Key Concepts

### The Three Pillars of Observability

**Metrics**
- Numeric measurements over time (CPU, memory, request count, latency)
- Aggregated data for trends and patterns
- CloudWatch Metrics, custom metrics, embedded metric format
- Use cases: Performance monitoring, capacity planning, auto-scaling triggers

**Logs**
- Discrete events with timestamp and context
- Unstructured or structured (JSON)
- CloudWatch Logs, S3, OpenSearch
- Use cases: Debugging, audit trail, security analysis, compliance

**Traces**
- End-to-end request flow through distributed systems
- Correlation IDs linking related operations
- AWS X-Ray for distributed tracing
- Use cases: Performance bottlenecks, dependency analysis, error diagnosis

### Amazon CloudWatch

**CloudWatch Metrics**
- **Standard Metrics**: 1-minute or 5-minute granularity (free for basic, paid for detailed)
- **Custom Metrics**: Application-specific metrics (PutMetricData API)
- **High-Resolution Metrics**: 1-second granularity (higher cost)
- **Dimensions**: Key-value pairs for filtering (InstanceId, Environment)
- **Statistics**: Sum, Average, Min, Max, SampleCount, percentiles
- **Metric Math**: Combine multiple metrics (CPU + Memory, rate of change)
- **Anomaly Detection**: ML-based detection of unusual patterns
- **Composite Alarms**: Combine multiple alarms with AND/OR logic

**CloudWatch Logs**
- **Log Groups**: Container for log streams
- **Log Streams**: Sequence of log events from single source
- **Retention**: 1 day to indefinite (default: never expire)
- **Encryption**: KMS encryption at rest
- **Metric Filters**: Extract metrics from logs
- **Subscription Filters**: Stream logs to Lambda, Kinesis, Firehose
- **Insights**: SQL-like query language for log analysis
- **Live Tail**: Real-time log viewing
- **Cross-Account Logging**: Centralize logs from multiple accounts

**CloudWatch Alarms**
- **Metric Alarms**: Threshold on metric (CPU > 80%)
- **Composite Alarms**: AND/OR logic across multiple alarms
- **States**: OK, ALARM, INSUFFICIENT_DATA
- **Actions**: SNS, Auto Scaling, EC2 actions (stop, terminate, reboot), Systems Manager
- **Evaluation Periods**: Number of periods to evaluate (3 out of 5 periods)
- **Datapoints to Alarm**: How many datapoints must breach threshold
- **Missing Data**: Treat as breaching, not breaching, good, or ignore

**CloudWatch Dashboards**
- Visual representation of metrics and logs
- Cross-region and cross-account
- Widget types: Line, stacked area, number, gauge, pie, bar, text, log table
- Automatic dashboards for AWS services
- Live data with auto-refresh

**CloudWatch Insights**
- **Container Insights**: ECS, EKS metrics and logs (CPU, memory, network, storage)
- **Lambda Insights**: Enhanced Lambda monitoring (cold starts, memory, duration)
- **Contributor Insights**: Identify top contributors (top IPs, URLs, users)
- **Application Insights**: Automatic dashboards for .NET and SQL Server

**CloudWatch Synthetics**
- **Canaries**: Scheduled scripts to monitor endpoints
- **Blueprints**: Heartbeat, API, Broken link checker, Visual monitoring
- **Runtime**: Node.js or Python
- **Screenshots**: Visual regression testing
- **Use Cases**: Uptime monitoring, UI testing, API availability

### AWS X-Ray

**Distributed Tracing**
- **Service Map**: Visual representation of service dependencies
- **Traces**: Request flow across services with timing
- **Segments**: Work done by single service
- **Subsegments**: Granular detail within segment (database query, external API call)
- **Annotations**: Key-value pairs for filtering (UserId, OrderId)
- **Metadata**: Additional data not used for filtering

**X-Ray Integration**
- **EC2**: X-Ray daemon on instance
- **ECS/EKS**: X-Ray sidecar container
- **Lambda**: Enable active tracing (built-in integration)
- **API Gateway**: Enable active tracing
- **Elastic Beanstalk**: Configuration option
- **App Runner**: Built-in support

**Sampling**
- **Reservoir**: Minimum traces per second (1 trace/second default)
- **Rate**: Percentage of additional requests (5% default)
- **Custom Rules**: Sampling based on service name, URL path, HTTP method

**Use Cases**
- Identify performance bottlenecks
- Understand service dependencies
- Analyze errors and exceptions
- Track user transactions end-to-end

### Amazon OpenSearch Service (Elasticsearch)

**Log Analytics**
- **Ingestion**: Kinesis Firehose, Logstash, Lambda, CloudWatch Logs subscription
- **Indexing**: Documents indexed for fast search
- **Search**: Full-text search, filtering, aggregations
- **Visualization**: OpenSearch Dashboards (Kibana)
- **Alerting**: Monitor and alert on log patterns
- **Cluster**: Master nodes, data nodes, UltraWarm nodes

**Use Cases**
- Centralized log management
- Application performance monitoring
- Security information and event management (SIEM)
- Business analytics

**UltraWarm Storage**
- S3-backed warm storage tier
- Lower cost than hot data nodes
- Slower query performance (acceptable for historical data)
- Lifecycle management (hot → warm → cold delete)

### AWS Distro for OpenTelemetry (ADOT)

**Open-Source Observability**
- **OpenTelemetry**: Vendor-neutral standard for telemetry (traces, metrics, logs)
- **ADOT Collector**: Receive, process, export telemetry
- **Destinations**: X-Ray, CloudWatch, Amazon Managed Prometheus, third-party (Datadog, New Relic)
- **Instrumentation**: Auto-instrumentation for common frameworks
- **Use Cases**: Multi-vendor observability, hybrid cloud, avoiding vendor lock-in

### Amazon Managed Grafana

**Visualization and Analytics**
- Managed Grafana service
- **Data Sources**: CloudWatch, X-Ray, Prometheus, OpenSearch, Timestream, Athena, Redshift
- **Dashboards**: Multi-source visualization
- **Alerting**: Alert based on queries
- **SAML/SSO**: Integration with corporate identity providers
- **Use Cases**: Unified dashboards across AWS and third-party, open-source tooling

### Amazon Managed Service for Prometheus

**Prometheus Metrics**
- Managed Prometheus-compatible service
- **Metrics**: Time-series metrics with labels
- **PromQL**: Query language for metrics
- **Remote Write**: Prometheus servers send metrics to managed service
- **Grafana Integration**: Visualize Prometheus metrics
- **Use Cases**: Container/Kubernetes monitoring, multi-cluster observability

### AWS Systems Manager

**Operational Insights**
- **OpsCenter**: Centralized operational issue management (OpsItems)
- **Explorer**: Dashboard for account resources and operational data
- **Inventory**: Collect metadata from instances (software, network, OS)
- **Compliance**: Check configuration compliance
- **Session Manager**: Secure shell access without SSH keys
- **Run Command**: Execute commands at scale
- **Patch Manager**: Automated patching

### Third-Party Integrations

**APM Tools**
- **Datadog**: Metrics, logs, traces, dashboards
- **New Relic**: Application performance monitoring
- **Dynatrace**: Full-stack monitoring
- **Splunk**: Log management and SIEM
- **Sumo Logic**: Cloud-native log analytics

**Integration Methods**
- CloudWatch Logs subscription filters → Kinesis → Third-party
- CloudWatch Metric Streams → Kinesis Firehose → Third-party
- X-Ray data export to third-party
- Agent-based collection directly to third-party

## AWS Services Reference

### Core Services

**Amazon CloudWatch**
- Metrics, logs, alarms, dashboards
- Container Insights, Lambda Insights
- Synthetics for endpoint monitoring
- Anomaly detection

**AWS X-Ray**
- Distributed tracing
- Service map
- Performance analysis
- Error tracking

**Amazon OpenSearch Service**
- Log aggregation and search
- Visualization with dashboards
- Alerting on log patterns
- SIEM capabilities

**AWS CloudTrail**
- API call logging
- Audit and compliance
- CloudTrail Insights for anomalies
- CloudTrail Lake for querying

### Supporting Services

**Amazon Managed Grafana**
- Multi-source visualization
- Dashboard as code
- Alerting and notifications
- SSO integration

**Amazon Managed Service for Prometheus**
- Prometheus-compatible metrics
- PromQL queries
- Container monitoring
- Multi-cluster aggregation

**AWS Distro for OpenTelemetry**
- Vendor-neutral telemetry
- ADOT Collector
- Multi-destination export
- Auto-instrumentation

**Amazon EventBridge**
- Event-driven automation
- CloudWatch Events replacement
- Schedule expressions
- Cross-account events

**AWS Systems Manager**
- OpsCenter for issue management
- Explorer for resource dashboards
- Inventory and compliance
- Session Manager for access

## Architecture Patterns

### Pattern 1: Centralized Logging Architecture

**Use Case**
- Multi-account organization (50 AWS accounts)
- Centralized log storage and analysis
- Compliance and audit requirements
- Security monitoring and alerting

**Implementation Approach**
1. **Log Sources**: CloudTrail, VPC Flow Logs, CloudWatch Logs from all accounts
2. **Centralization**:
   - CloudWatch Logs subscription filters → Kinesis Data Firehose
   - Kinesis Firehose → S3 (centralized log archive account)
   - Optional: Kinesis Firehose → OpenSearch for real-time analysis
3. **S3 Bucket**:
   - Partitioned by account, region, service, date
   - Lifecycle policy: Standard → IA (90 days) → Glacier (1 year)
   - Object Lock for immutability (compliance mode, 7 years)
4. **Analysis**:
   - Athena for ad-hoc SQL queries on S3
   - OpenSearch for real-time search and dashboards
   - QuickSight for business intelligence
5. **Security**:
   - KMS encryption (separate keys per account)
   - IAM roles for cross-account access
   - VPC endpoints for private connectivity
6. **Alerting**: OpenSearch alerts or CloudWatch alarms on metric filters

**Pros/Cons**
- Pros: Centralized visibility, compliance, cost-effective storage, queryable with SQL
- Cons: Near real-time (buffer delay), complexity, cross-account IAM

### Pattern 2: Distributed Tracing for Microservices

**Use Case**
- Microservices architecture with 50 services
- Identify performance bottlenecks
- Understand service dependencies
- Troubleshoot errors across service boundaries

**Implementation Approach**
1. **Instrumentation**: X-Ray SDK in all services (Node.js, Python, Java, .NET)
2. **Correlation**: Trace ID propagated through headers (X-Amzn-Trace-Id)
3. **Collection**:
   - Lambda: Active tracing enabled (built-in)
   - ECS: X-Ray daemon as sidecar container
   - API Gateway: Active tracing enabled
4. **Service Map**: Automatic visualization of service dependencies
5. **Trace Analysis**:
   - Filter by annotation (UserId, OrderId, Environment)
   - Analyze latency distribution (p50, p90, p99)
   - Identify failed requests and root cause
6. **Sampling**: Custom sampling rules (100% for errors, 5% for success)
7. **Alerting**: CloudWatch alarms on X-Ray metrics (error rate, latency)
8. **Integration**: X-Ray data to ADOT Collector → third-party APM

**Pros/Cons**
- Pros: End-to-end visibility, identify bottlenecks, minimal code changes, automatic service map
- Cons: Sampling (not 100% traces), X-Ray daemon overhead, cost at high scale

### Pattern 3: Container Monitoring with Container Insights

**Use Case**
- EKS cluster with 100 nodes and 500 pods
- Monitor CPU, memory, network, storage
- Identify resource-constrained pods
- Detect anomalies and trends

**Implementation Approach**
1. **Enable Container Insights**: CloudWatch agent as DaemonSet on EKS
2. **Metrics**:
   - Cluster: CPU, memory, network, disk
   - Namespace: Pod count, resource utilization
   - Node: CPU, memory, network, disk
   - Pod: CPU, memory, network, storage
   - Container: CPU, memory
3. **Performance Insights**: Automatic dashboards in CloudWatch
4. **Logs**: Container stdout/stderr → CloudWatch Logs
5. **Prometheus Integration**: CloudWatch agent scrapes Prometheus metrics
6. **Alerting**: CloudWatch alarms for high CPU, memory, pod restart count
7. **Cost Optimization**: Identify over-provisioned pods, right-size resources
8. **Grafana**: Amazon Managed Grafana for custom dashboards

**Pros/Cons**
- Pros: Automatic metrics collection, pre-built dashboards, integrated with CloudWatch ecosystem
- Cons: CloudWatch Logs costs, metrics storage costs, 1-minute granularity

### Pattern 4: Application Performance Monitoring (APM)

**Use Case**
- E-commerce web application (monolith and microservices)
- Monitor application performance (response time, error rate, throughput)
- User experience monitoring
- Proactive alerting

**Implementation Approach**
1. **Backend Monitoring**:
   - X-Ray for distributed tracing
   - CloudWatch custom metrics (business metrics: orders/minute, revenue/hour)
   - Lambda Insights for serverless functions
2. **Frontend Monitoring**:
   - CloudWatch RUM (Real User Monitoring) for browser performance
   - Metrics: Page load time, JavaScript errors, user sessions
3. **Synthetics**:
   - CloudWatch Synthetics canaries for endpoint availability
   - Heartbeat canary every 1 minute
   - API canary for critical endpoints
   - Visual monitoring for UI regression
4. **Custom Metrics**:
   - Embedded Metric Format (EMF) in application logs
   - Business KPIs (conversion rate, cart abandonment)
5. **Dashboards**: CloudWatch dashboards with application, infrastructure, business metrics
6. **Alerting**:
   - Error rate > 1% → SNS → PagerDuty
   - Latency p99 > 1s → SNS → Slack
   - Canary failure → SNS → on-call engineer
7. **Anomaly Detection**: CloudWatch anomaly detection for unusual patterns

**Pros/Cons**
- Pros: Full-stack visibility, proactive alerting, custom business metrics, native AWS integration
- Cons: Multiple tools to learn, costs accumulate, CloudWatch Logs/RUM costs

### Pattern 5: Security Monitoring and SIEM

**Use Case**
- Financial services with compliance requirements (SOC 2, PCI-DSS)
- Real-time security event detection
- Centralized security monitoring
- Forensic investigation capabilities

**Implementation Approach**
1. **Data Sources**:
   - CloudTrail (API activity)
   - VPC Flow Logs (network traffic)
   - GuardDuty findings
   - Security Hub findings
   - WAF logs
   - Config compliance events
2. **Ingestion**:
   - CloudWatch Logs → Subscription filter → Kinesis Firehose
   - S3 (CloudTrail, VPC Flow Logs, WAF) → OpenSearch
3. **OpenSearch**:
   - Real-time indexing and search
   - Security dashboards (failed logins, unusual API calls, network anomalies)
   - Alerting on suspicious patterns
4. **Automated Response**:
   - GuardDuty finding → EventBridge → Lambda → Remediation
   - Example: Compromised EC2 → Isolate (replace security group) → Snapshot → Notify
5. **Forensics**:
   - Immutable logs in S3 with Object Lock
   - CloudTrail log file validation
   - Athena for historical investigation
6. **Compliance**: Audit Manager for continuous compliance evidence collection
7. **Third-Party SIEM**: Optionally integrate with Splunk, Sumo Logic, QRadar

**Pros/Cons**
- Pros: Real-time detection, automated response, centralized visibility, compliance
- Cons: OpenSearch costs, complexity, alert fatigue (tuning required)

### Pattern 6: Multi-Cloud Observability with OpenTelemetry

**Use Case**
- Hybrid and multi-cloud environment (AWS, Azure, on-premises)
- Vendor-neutral observability
- Unified dashboards across environments
- Flexibility to change APM vendors

**Implementation Approach**
1. **Instrumentation**: OpenTelemetry SDKs in applications (Java, Python, Node.js, Go)
2. **ADOT Collector**:
   - AWS: Deployed as sidecar (ECS, EKS) or Lambda extension
   - On-premises: Standalone collector
3. **Telemetry Export**:
   - Traces → AWS X-Ray and/or Datadog
   - Metrics → Amazon Managed Prometheus and/or Datadog
   - Logs → CloudWatch Logs and/or Splunk
4. **Visualization**:
   - Amazon Managed Grafana with Prometheus, CloudWatch, X-Ray data sources
   - Dashboards showing AWS and non-AWS resources
5. **Alerting**: Grafana alerting based on PromQL queries
6. **Flexibility**: Change export destinations without changing application code

**Pros/Cons**
- Pros: Vendor-neutral, multi-cloud support, flexibility, open-source standard
- Cons: Additional complexity, ADOT Collector overhead, learning curve

## Best Practices

### Enterprise-Level Recommendations

**Comprehensive Observability**
- Implement all three pillars: metrics, logs, traces
- Collect metrics at infrastructure, application, and business levels
- Centralize logs from all accounts and regions
- Enable distributed tracing for microservices

**Alerting Strategy**
- Alert on symptoms (customer impact) not causes (CPU high)
- Use composite alarms to reduce noise
- Implement escalation (warning → critical)
- Integrate with incident management (PagerDuty, Opsgenie)
- Define SLIs, SLOs, SLAs and alert on SLO violations

**Dashboard Design**
- Executive dashboard: Business KPIs, SLA compliance
- Operations dashboard: Infrastructure health, alerts
- Application dashboard: Performance metrics, errors, throughput
- Use consistent color schemes (green = good, red = bad)
- Auto-refresh for real-time monitoring

**Data Retention**
- Short-term (7-30 days): High-resolution metrics, detailed logs
- Medium-term (90 days - 1 year): Standard metrics, aggregated logs
- Long-term (1-7 years): Compliance logs in S3 Glacier
- Balance retention with cost

**Cost Optimization**
- Metric filters instead of ingesting all logs to CloudWatch
- Aggregation at source (reduce log volume)
- S3 for long-term log storage (cheaper than CloudWatch)
- Sampling for traces (not 100%)
- Right-size log retention based on compliance requirements

### Security Considerations

**Encryption**
- CloudWatch Logs: KMS encryption
- S3 log storage: SSE-KMS
- In-transit: TLS for all log shipping

**Access Control**
- IAM policies for CloudWatch, X-Ray, OpenSearch
- Cross-account roles for centralized logging
- Principle of least privilege
- Separate read-only roles for engineers

**Audit Trail**
- CloudTrail for observability service usage
- Who viewed logs, created dashboards, modified alarms
- Immutable logs for forensics

**Sensitive Data**
- Avoid logging PII, passwords, API keys
- Redaction or masking in logs
- Separate log streams for sensitive vs. non-sensitive

### Performance Tuning

**CloudWatch Metrics**
- Use metric math instead of multiple GetMetricStatistics API calls
- Batch PutMetricData calls (up to 20 metrics per call)
- High-resolution metrics only when needed (higher cost)

**CloudWatch Logs**
- Subscription filters with patterns (don't stream all logs)
- Batch log events in PutLogEvents (up to 1 MB)
- Use Kinesis Data Firehose buffer for cost efficiency

**X-Ray**
- Sampling rules to control cost and overhead
- Asynchronous segment submission
- Batch segment submission

**OpenSearch**
- Dedicated master nodes for cluster stability
- Data node sizing based on index size and query load
- UltraWarm for historical data (50-70% cost savings)
- Index lifecycle management (hot → warm → delete)

## Common Scenarios

### Scenario 1: E-Commerce Application Observability

**Context**: E-commerce platform with 10M daily users, microservices architecture (50 services)

**Observability Stack**
1. **Metrics**:
   - **Infrastructure**: CloudWatch default metrics (EC2, ALB, RDS, DynamoDB)
   - **Application**: Custom metrics (orders/minute, revenue/hour, cart abandonment rate)
   - **Business**: Conversion rate, average order value, customer lifetime value
2. **Logs**:
   - **Application Logs**: CloudWatch Logs from ECS containers
   - **Access Logs**: ALB access logs → S3
   - **Database Logs**: RDS slow query logs → CloudWatch Logs
3. **Traces**:
   - **X-Ray**: Distributed tracing across API Gateway, Lambda, ECS
   - **Annotations**: OrderId, UserId, ProductId for filtering
4. **Synthetics**:
   - **Canaries**: Every 5 minutes check homepage, product search, checkout
5. **Dashboards**:
   - **Executive**: Revenue, orders, conversion rate (real-time)
   - **Operations**: Infrastructure health, error rates, latency
   - **Engineering**: Service map, trace analysis, logs
6. **Alerting**:
   - **Critical**: Checkout errors > 1% (immediate page)
   - **Warning**: Latency p99 > 2s (Slack notification)
   - **Info**: Daily summary of KPIs (email)

**Key Considerations**
- Embedded Metric Format (EMF) for custom metrics in logs
- Trace 100% of checkout flows, 5% of browse flows (sampling)
- CloudWatch Logs Insights for debugging specific user issues
- X-Ray service map to identify new dependencies during deployments

### Scenario 2: Kubernetes Cluster Monitoring

**Context**: EKS cluster with 50 nodes, 200 deployments, 500 pods

**Observability Stack**
1. **Container Insights**: CloudWatch agent DaemonSet on EKS
   - Metrics: Cluster, node, namespace, pod, container levels
2. **Prometheus**: Prometheus server in EKS scraping pod metrics
   - Export to Amazon Managed Prometheus
3. **Logs**: Fluent Bit DaemonSet → CloudWatch Logs
   - Container stdout/stderr
   - Kubernetes events
4. **Tracing**: ADOT Collector sidecar → X-Ray
5. **Dashboards**: Amazon Managed Grafana
   - CloudWatch data source for Container Insights
   - Prometheus data source for custom metrics
   - Unified view of cluster health
6. **Alerting**:
   - **Critical**: Node NotReady, Pod CrashLoopBackOff
   - **Warning**: High CPU/memory, disk pressure
   - **Capacity**: Cluster CPU > 70% (add nodes)
7. **Cost Monitoring**: Kubecost for pod-level cost allocation

**Key Considerations**
- Container Insights for automatic metrics, Prometheus for custom app metrics
- Separate log groups per namespace for access control
- X-Ray sampling (100% errors, 10% success)
- Grafana alerts based on PromQL queries

### Scenario 3: Serverless Application Monitoring

**Context**: Serverless API with API Gateway, Lambda (100 functions), DynamoDB

**Observability Stack**
1. **Metrics**:
   - **API Gateway**: Request count, 4xx/5xx errors, latency
   - **Lambda**: Invocations, errors, duration, throttles, concurrent executions
   - **DynamoDB**: Consumed capacity, throttled requests, latency
   - **Custom**: Business metrics (signups/hour, transactions/minute)
2. **Logs**:
   - **Lambda Logs**: CloudWatch Logs (automatic)
   - **API Gateway Logs**: Execution logs and access logs
   - **Embedded Metric Format**: Custom metrics in Lambda logs
3. **Tracing**:
   - **Lambda Active Tracing**: X-Ray enabled on all functions
   - **API Gateway Active Tracing**: End-to-end request tracing
   - **DynamoDB X-Ray Integration**: Database call tracing
4. **Lambda Insights**: Enhanced metrics for cold starts, memory usage
5. **CloudWatch RUM**: Frontend JavaScript errors and performance
6. **Dashboards**: CloudWatch dashboards with API, Lambda, DynamoDB metrics
7. **Alerting**:
   - **Critical**: Lambda errors > 1%, DynamoDB throttling
   - **Warning**: Lambda duration approaching timeout, cold starts > 10%
   - **Capacity**: Lambda concurrent executions > 80% of limit

**Key Considerations**
- Lambda Insights for cold start analysis and memory optimization
- X-Ray subsegments for DynamoDB queries (identify slow queries)
- EMF for custom business metrics (no PutMetricData API call overhead)
- CloudWatch Logs Insights for debugging specific transactions

### Scenario 4: Database Performance Monitoring

**Context**: RDS PostgreSQL and DynamoDB for multi-tier application

**Observability Stack**
1. **RDS**:
   - **Performance Insights**: Database load, top SQL, wait events
   - **Enhanced Monitoring**: OS-level metrics (1-second granularity)
   - **CloudWatch Metrics**: CPU, connections, IOPS, latency, replication lag
   - **Slow Query Logs**: CloudWatch Logs
2. **DynamoDB**:
   - **CloudWatch Metrics**: Consumed capacity, throttled requests, latency
   - **Contributor Insights**: Top partition keys by traffic
   - **X-Ray Integration**: Query-level tracing
3. **Dashboards**: Combined RDS and DynamoDB metrics
4. **Alerting**:
   - **RDS**: CPU > 80%, replication lag > 60s, connections > 80% of max
   - **DynamoDB**: Throttled requests > 10, latency p99 > 100ms
5. **Optimization**:
   - Performance Insights to identify slow queries
   - Contributor Insights to detect hot partitions
   - CloudWatch Logs Insights for slow query analysis
   - Recommendations from RDS Performance Insights

**Key Considerations**
- Performance Insights free tier (7 days retention, 1-hour granularity)
- Enhanced Monitoring for OS-level troubleshooting (disk I/O wait)
- DynamoDB Contributor Insights for hot partition detection
- Combine RDS and application metrics for correlation

### Scenario 5: Security Incident Investigation

**Context**: Suspicious GuardDuty finding for unauthorized EC2 instance access

**Investigation Process**
1. **Initial Alert**: GuardDuty → EventBridge → SNS → Security team
2. **GuardDuty Finding Details**:
   - Finding type: UnauthorizedAccess:EC2/SSHBruteForce
   - Affected instance: i-1234567890abcdef0
   - Source IP: 203.0.113.100 (outside organization)
   - Timestamp: 2025-01-10 14:32:15 UTC
3. **CloudTrail Investigation**:
   - Athena query: Who launched the instance? When? With what IAM role?
   - Result: IAM user "contractor-john", launched 2 days ago
4. **VPC Flow Logs**:
   - Athena query: All connections to instance in last 7 days
   - Result: Multiple SSH attempts from 203.0.113.100, all rejected
5. **CloudWatch Logs**:
   - /var/log/auth.log from instance (shipped to CloudWatch)
   - Result: Failed SSH login attempts, no successful logins
6. **X-Ray**: No traces found (instance not running instrumented applications)
7. **Remediation**:
   - Lambda triggered by EventBridge: Isolate instance (replace security group)
   - Snapshot EBS for forensics
   - Revoke "contractor-john" IAM credentials
   - Notify security team for manual investigation
8. **Post-Incident**:
   - Review CloudTrail for all "contractor-john" actions
   - Add GuardDuty suppression rule if determined to be false positive
   - Update incident playbook in Systems Manager Automation

**Key Considerations**
- Centralized logging enables correlation across services
- CloudTrail for "who did what"
- VPC Flow Logs for "what network traffic occurred"
- Automated response limits blast radius
- Immutable logs (S3 Object Lock) prevent tampering

## AWS CLI Examples

```bash
# CloudWatch Metrics - Put custom metric
aws cloudwatch put-metric-data \
  --namespace "ECommerce/Orders" \
  --metric-name OrdersPerMinute \
  --value 150 \
  --timestamp 2025-01-10T12:00:00Z \
  --dimensions Environment=Production,Region=US-East

# CloudWatch Metrics - Get metric statistics
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --start-time 2025-01-10T00:00:00Z \
  --end-time 2025-01-10T23:59:59Z \
  --period 300 \
  --statistics Average,Maximum \
  --query 'Datapoints[].[Timestamp,Average,Maximum]' \
  --output table

# CloudWatch Alarms - Create metric alarm
aws cloudwatch put-metric-alarm \
  --alarm-name high-cpu-alarm \
  --alarm-description "Alert when CPU exceeds 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --evaluation-periods 2 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --datapoints-to-alarm 2 \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:alert-topic

# CloudWatch Alarms - Create composite alarm
aws cloudwatch put-composite-alarm \
  --alarm-name critical-application-alarm \
  --alarm-description "Critical when high CPU AND high latency" \
  --alarm-rule "ALARM(high-cpu-alarm) AND ALARM(high-latency-alarm)" \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:critical-alerts

# CloudWatch Logs - Create log group
aws logs create-log-group \
  --log-group-name /aws/lambda/order-processing \
  --kms-key-id arn:aws:kms:us-east-1:123456789012:key/12345678 \
  --tags Environment=Production,Application=ECommerce

# CloudWatch Logs - Set retention
aws logs put-retention-policy \
  --log-group-name /aws/lambda/order-processing \
  --retention-in-days 30

# CloudWatch Logs - Create metric filter
aws logs put-metric-filter \
  --log-group-name /aws/lambda/order-processing \
  --filter-name ErrorCount \
  --filter-pattern "[time, request_id, level=ERROR, msg]" \
  --metric-transformations \
    metricName=ApplicationErrors,\
metricNamespace=ECommerce/Lambda,\
metricValue=1,\
defaultValue=0

# CloudWatch Logs Insights - Query logs
aws logs start-query \
  --log-group-name /aws/lambda/order-processing \
  --start-time $(date -u -d '1 hour ago' +%s) \
  --end-time $(date -u +%s) \
  --query-string 'fields @timestamp, @message | filter @message like /ERROR/ | sort @timestamp desc | limit 20'

# CloudWatch Logs - Create subscription filter
aws logs put-subscription-filter \
  --log-group-name /aws/lambda/order-processing \
  --filter-name ship-to-opensearch \
  --filter-pattern "" \
  --destination-arn arn:aws:firehose:us-east-1:123456789012:deliverystream/logs-to-opensearch

# CloudWatch Dashboards - Create dashboard
aws cloudwatch put-dashboard \
  --dashboard-name production-overview \
  --dashboard-body file://dashboard.json

# X-Ray - Get service graph
aws xray get-service-graph \
  --start-time $(date -u -d '1 hour ago' +%s) \
  --end-time $(date -u +%s)

# X-Ray - Get trace summaries
aws xray get-trace-summaries \
  --start-time $(date -u -d '1 hour ago' +%s) \
  --end-time $(date -u +%s) \
  --filter-expression 'service("order-service") AND http.status = 500' \
  --query 'TraceSummaries[].[Id,Duration,Http.HttpStatus]' \
  --output table

# X-Ray - Get trace details
aws xray batch-get-traces \
  --trace-ids <trace-id-1> <trace-id-2>

# X-Ray - Create sampling rule
aws xray create-sampling-rule \
  --cli-input-json file://sampling-rule.json

# sampling-rule.json:
# {
#   "SamplingRule": {
#     "RuleName": "ErrorTracing",
#     "Priority": 1000,
#     "FixedRate": 1.0,
#     "ReservoirSize": 1,
#     "ServiceName": "*",
#     "ServiceType": "*",
#     "Host": "*",
#     "HTTPMethod": "*",
#     "URLPath": "*",
#     "Version": 1,
#     "Attributes": {
#       "http.status": "5*"
#     }
#   }
# }

# CloudWatch Synthetics - Create canary
aws synthetics create-canary \
  --name homepage-check \
  --code file://canary-code.zip \
  --artifact-s3-location s3://canary-artifacts/ \
  --execution-role-arn arn:aws:iam::123456789012:role/CanaryExecutionRole \
  --schedule Expression='rate(5 minutes)' \
  --runtime-version syn-nodejs-puppeteer-6.0 \
  --run-config TimeoutInSeconds=60,MemoryInMB=960 \
  --success-retention-period 31 \
  --failure-retention-period 31

# CloudWatch Synthetics - Start canary
aws synthetics start-canary --name homepage-check

# CloudWatch Contributor Insights - Create rule
aws cloudwatch put-insight-rule \
  --rule-name TopIPAddresses \
  --rule-state ENABLED \
  --rule-definition file://insight-rule.json

# insight-rule.json:
# {
#   "Schema": {"Name": "CloudWatchLogRule","Version": 1},
#   "LogGroupNames": ["/aws/apigateway/access-logs"],
#   "LogFormat": "JSON",
#   "Contribution": {
#     "Keys": ["$.sourceIp"],
#     "Filters": [{"Match": "$.status","GreaterThan": 399}]
#   },
#   "AggregateOn": "Count"
# }

# CloudWatch Anomaly Detection - Create anomaly detector
aws cloudwatch put-anomaly-detector \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --stat Average

# CloudWatch Anomaly Detection - Create alarm based on anomaly
aws cloudwatch put-metric-alarm \
  --alarm-name cpu-anomaly-alarm \
  --comparison-operator LessThanLowerOrGreaterThanUpperThreshold \
  --evaluation-periods 2 \
  --metrics '[
    {
      "Id": "m1",
      "ReturnData": true,
      "MetricStat": {
        "Metric": {"Namespace":"AWS/EC2","MetricName":"CPUUtilization","Dimensions":[{"Name":"InstanceId","Value":"i-1234567890abcdef0"}]},
        "Period": 300,
        "Stat": "Average"
      }
    },
    {
      "Id": "ad1",
      "Expression": "ANOMALY_DETECTION_BAND(m1, 2)",
      "Label": "CPUUtilization (expected)"
    }
  ]' \
  --threshold-metric-id ad1 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:anomaly-alerts

# Systems Manager OpsCenter - Create OpsItem
aws ssm create-ops-item \
  --title "High CPU usage on production web servers" \
  --description "CPU usage exceeded 90% on 3 web servers" \
  --priority 2 \
  --source "CloudWatch Alarm" \
  --operational-data '{"CloudWatchAlarm":{"Value":"high-cpu-alarm","Type":"SearchableString"}}'

# Systems Manager OpsCenter - Update OpsItem
aws ssm update-ops-item \
  --ops-item-id oi-1234567890abcdef \
  --status InProgress \
  --notifications Arn=arn:aws:sns:us-east-1:123456789012:ops-notifications
```

## Study Tips

### SAP-C02 Exam Focus Areas

**High-Priority Topics**
- CloudWatch Metrics, Logs, Alarms (comprehensive understanding)
- CloudWatch Logs Insights queries
- X-Ray distributed tracing for microservices
- Container Insights for ECS/EKS
- Lambda Insights for serverless
- Centralized logging patterns
- CloudWatch Synthetics for endpoint monitoring
- Metric filters and custom metrics
- Composite alarms

**Scenario-Based Questions**
- Design centralized logging architecture
- Choose appropriate monitoring service based on requirements
- Implement distributed tracing for microservices
- Create custom metrics for business KPIs
- Set up alerting with appropriate thresholds
- Troubleshoot performance issues using observability data
- Cost optimization for monitoring at scale

**Common Decision Points**
- **CloudWatch vs. Third-Party**: Native integration vs. advanced features
- **Metrics vs. Logs**: Aggregated data vs. detailed events
- **CloudWatch Logs vs. S3**: Real-time vs. cost-effective storage
- **X-Ray vs. Third-Party APM**: AWS-native vs. multi-cloud
- **Standard vs. High-Resolution Metrics**: Cost vs. granularity
- **CloudWatch vs. OpenSearch**: Simple vs. advanced search/analytics

### Key Differences from SAA-C03

**SAA-C03 Knowledge**
- Basic CloudWatch metrics and alarms
- CloudWatch Logs fundamentals
- Simple dashboards

**Additional SAP-C02 Requirements**
- CloudWatch Logs Insights query language
- Centralized logging across multiple accounts
- X-Ray service maps and trace analysis
- Container Insights and Lambda Insights
- CloudWatch Synthetics for synthetic monitoring
- Composite alarms and anomaly detection
- Embedded Metric Format (EMF)
- CloudWatch Contributor Insights
- Integration with third-party APM tools
- OpenSearch for log analytics
- Amazon Managed Grafana and Prometheus
- AWS Distro for OpenTelemetry (ADOT)
- Observability cost optimization strategies

### Complex Scenarios to Master

**Centralized Multi-Account Logging**
- CloudWatch Logs subscription filters
- Cross-account log aggregation
- Kinesis Firehose for delivery
- S3 partitioning and lifecycle
- Athena for querying

**Distributed Tracing**
- X-Ray instrumentation in multiple services
- Correlation IDs and trace propagation
- Service map interpretation
- Trace filtering and analysis
- Sampling strategies

**Container Observability**
- Container Insights setup on EKS
- Prometheus metrics collection
- Fluent Bit for log shipping
- Grafana dashboards
- Resource optimization based on metrics

**Custom Metrics and Business KPIs**
- PutMetricData API
- Embedded Metric Format in logs
- Metric math for derived metrics
- Dashboards combining technical and business metrics

**Security Monitoring**
- CloudTrail, VPC Flow Logs, GuardDuty integration
- OpenSearch for SIEM
- Automated incident response
- Forensic investigation

### Practice Lab Recommendations

1. **CloudWatch Metrics and Alarms**
   - Create custom metrics with PutMetricData
   - Create metric alarm with SNS notification
   - Test alarm by triggering threshold
   - Create composite alarm
   - Configure anomaly detection

2. **CloudWatch Logs Insights**
   - Ship application logs to CloudWatch Logs
   - Write Insights queries for error analysis
   - Create metric filter from logs
   - Set up subscription filter to Kinesis

3. **X-Ray Distributed Tracing**
   - Instrument Lambda functions with X-Ray
   - Enable X-Ray on API Gateway
   - Generate traces by invoking API
   - Analyze service map
   - Filter traces by annotation

4. **Container Insights on EKS**
   - Deploy CloudWatch agent on EKS
   - View Container Insights dashboards
   - Create alarms on pod metrics
   - Analyze pod resource usage

5. **Centralized Logging**
   - Create CloudWatch Logs in multiple accounts
   - Set up subscription filters to Kinesis
   - Deliver to S3 with Firehose
   - Query with Athena

6. **CloudWatch Synthetics**
   - Create heartbeat canary
   - Create API canary
   - Test failure scenarios
   - View canary metrics and screenshots

## Additional Resources

### AWS Whitepapers
- Operational Excellence Pillar - Well-Architected Framework
- Observability Best Practices on AWS
- Monitoring and Logging on AWS
- Debugging Distributed Systems on AWS

### Documentation Links
- Amazon CloudWatch: https://docs.aws.amazon.com/cloudwatch/
- AWS X-Ray: https://docs.aws.amazon.com/xray/
- Amazon OpenSearch Service: https://docs.aws.amazon.com/opensearch-service/
- AWS Distro for OpenTelemetry: https://aws-otel.github.io/
- Amazon Managed Grafana: https://docs.aws.amazon.com/grafana/
- Amazon Managed Service for Prometheus: https://docs.aws.amazon.com/prometheus/

### Video Resources
- AWS re:Invent sessions (search "OPN", "DEV" tracks for observability)
- CloudWatch Deep Dive
- X-Ray for Distributed Applications
- Container Insights for EKS
- Observability on AWS
