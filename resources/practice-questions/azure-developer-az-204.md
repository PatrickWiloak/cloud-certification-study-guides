# Azure Developer Associate (AZ-204) Practice Questions

40 scenario-based questions weighted by exam domain.

## Exam Domain Breakdown
| Domain | Weight | Questions |
|--------|--------|-----------|
| Develop Azure compute solutions | 25-30% | 11 |
| Develop for Azure storage | 15-20% | 7 |
| Implement Azure security | 20-25% | 9 |
| Monitor, troubleshoot, and optimize | 15-20% | 7 |
| Connect to and consume Azure services | 15-20% | 6 |

---

## Domain 1: Develop Azure Compute Solutions (Questions 1-11)

### Question 1
**Scenario:** A developer needs to create an Azure Function that processes messages from an Azure Service Bus queue. The function should scale automatically based on queue depth and handle message failures with retries. What trigger and configuration should they use?

A. HTTP trigger with manual queue polling
B. Service Bus Queue trigger with host.json retry configuration
C. Timer trigger that checks queue periodically
D. Event Grid trigger

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Service Bus Queue trigger automatically scales based on queue depth (up to 16 instances by default). The trigger handles message completion/abandonment. host.json configures retry policies for failed processing. HTTP trigger (A) requires manual polling. Timer trigger (C) isn't event-driven. Event Grid (D) is for events, not queue messages.

**Key Concept:** [Service Bus Trigger](https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-service-bus-trigger)
</details>

### Question 2
**Scenario:** A developer needs to deploy a containerized web application to Azure. The application should automatically scale based on HTTP traffic, support multiple revisions for blue-green deployments, and integrate with Dapr for service-to-service communication. What platform should they use?

A. Azure Kubernetes Service
B. Azure Container Apps
C. Azure Container Instances
D. Azure App Service

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Container Apps provides serverless containers with built-in HTTP autoscaling, revision-based traffic splitting for blue-green deployments, and native Dapr integration. AKS (A) requires manual Dapr installation and cluster management. ACI (C) doesn't have traffic splitting. App Service (D) has limited container features and no Dapr integration.

**Key Concept:** [Azure Container Apps](https://learn.microsoft.com/en-us/azure/container-apps/overview)
</details>

### Question 3
**Scenario:** A developer is creating an Azure Function that needs to output to multiple Azure services: write to Blob Storage, send a message to a queue, and update a Cosmos DB document. How should they implement this efficiently?

A. Use separate SDK calls in the function code for each service
B. Use multiple output bindings declared in function.json or attributes
C. Create separate functions for each output
D. Use Durable Functions for orchestration

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Output bindings allow declarative definition of outputs without SDK code. Multiple bindings can be defined for a single function (Blob, Queue, Cosmos DB outputs). The runtime handles connections and writes. SDK calls (A) work but require more code. Separate functions (C) add latency. Durable Functions (D) are for complex orchestrations.

**Key Concept:** [Function Bindings](https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-expressions-patterns)
</details>

### Question 4
**Scenario:** A developer needs to implement a long-running workflow that coordinates multiple Azure Functions, waits for human approval, and retries failed steps. What should they use?

A. Multiple timer-triggered functions with shared state in storage
B. Durable Functions with orchestrator, activity functions, and external events
C. Logic Apps
D. Azure Batch

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Durable Functions provide orchestration patterns including function chaining, fan-out/fan-in, human interaction (external events for approval), and automatic retry. Orchestrator functions maintain state. Timer functions (A) are complex to coordinate. Logic Apps (C) are low-code, not developer-focused. Batch (D) is for parallel processing, not workflows.

**Key Concept:** [Durable Functions](https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview)
</details>

### Question 5
**Scenario:** A developer needs to deploy an API to Azure App Service with zero-downtime deployments. They want to test new versions with a subset of traffic before full rollout. What should they configure?

A. Delete and redeploy the App Service
B. Deployment slots with traffic routing percentages
C. Multiple App Services behind Load Balancer
D. Azure DevOps release gates

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Deployment slots allow deploying new versions to a non-production slot, warming up the app, then gradually routing traffic (e.g., 10% to staging slot). Swap operation provides instant cutover with zero downtime. Delete/redeploy (A) causes downtime. Multiple services (C) require manual traffic management. Release gates (D) are for approval workflows, not traffic routing.

**Key Concept:** [Deployment Slots](https://learn.microsoft.com/en-us/azure/app-service/deploy-staging-slots)
</details>

### Question 6
**Scenario:** An Azure Function occasionally fails due to transient errors when calling an external API. The developer wants to implement automatic retries with exponential backoff. What should they configure?

A. Wrap the code in a try-catch with Thread.Sleep
B. Configure retry policy in host.json for the specific trigger type
C. Use Polly library for HTTP retries only
D. Create multiple function instances

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** host.json retry policies configure automatic retries with fixed delay, exponential backoff, or custom strategies at the trigger level. The runtime handles retry logic. Try-catch with Sleep (A) blocks the function. Polly (C) works for HTTP calls but host.json handles all trigger types. Multiple instances (D) don't address retries.

**Key Concept:** [Function Retry Policies](https://learn.microsoft.com/en-us/azure/azure-functions/functions-bindings-error-pages)
</details>

### Question 7
**Scenario:** A developer is building a web API using Azure App Service. They need to implement caching to reduce database load. Cached data should be shared across all instances of the App Service. What caching solution should they use?

A. In-memory caching in the application
B. Azure Cache for Redis
C. Output caching in App Service
D. Browser caching headers

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Cache for Redis provides a distributed cache shared across all App Service instances. It persists across instance restarts and scales independently. In-memory caching (A) isn't shared between instances. Output caching (C) is limited to response caching. Browser caching (D) is client-side only.

**Key Concept:** [Azure Cache for Redis](https://learn.microsoft.com/en-us/azure/azure-cache-for-redis/cache-overview)
</details>

### Question 8
**Scenario:** A developer needs to build an application that processes images uploaded to Blob Storage. Processing takes 2-3 minutes per image. The solution should scale automatically and be cost-effective for sporadic uploads.

A. App Service WebJob with always-on
B. Azure Function with Blob trigger on Consumption plan
C. Azure Function with Blob trigger on Premium plan
D. VM with polling application

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Blob trigger processes new blobs automatically. Premium plan provides pre-warmed instances (no cold starts), longer execution timeout (unlimited vs 10 min consumption), and VNET integration if needed. Consumption plan (B) has 10-minute timeout which may not be enough for 2-3 minute processing with retries. WebJobs (A) and VMs (D) require more management.

**Key Concept:** [Functions Premium Plan](https://learn.microsoft.com/en-us/azure/azure-functions/functions-premium-plan)
</details>

### Question 9
**Scenario:** A developer is containerizing an application for Azure Container Instances. The container needs to access secrets stored in Azure Key Vault at startup. How should they securely provide the secrets?

A. Pass secrets as environment variables in the deployment
B. Mount Key Vault secrets as a volume using managed identity
C. Embed secrets in the container image
D. Use Azure App Configuration

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** ACI can mount Key Vault secrets as files in a volume using managed identity for authentication. Secrets are fetched at container start without exposing them in configuration. Environment variables (A) are visible in container settings. Embedded secrets (C) are in the image layer. App Configuration (D) isn't for secrets.

**Key Concept:** [ACI Secret Volumes](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-volume-secret)
</details>

### Question 10
**Scenario:** A developer needs to configure an Azure Function to use a custom Docker container. The function should scale based on queue length. What hosting plan and configuration are required?

A. Consumption plan with container
B. Premium plan or Dedicated plan with custom container
C. Any plan supports containers
D. Only ACI supports custom containers

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Custom containers for Azure Functions require Premium plan or Dedicated (App Service) plan. Consumption plan doesn't support custom containers. Premium plan provides event-driven scaling. The container must include the Azure Functions runtime.

**Key Concept:** [Custom Containers](https://learn.microsoft.com/en-us/azure/azure-functions/functions-create-function-linux-custom-image)
</details>

### Question 11
**Scenario:** A developer needs to implement a fan-out/fan-in pattern where 100 tasks run in parallel and results are aggregated. Each task calls an external API. What Durable Functions pattern should they use?

A. Function chaining
B. Fan-out/fan-in with Task.WhenAll
C. Async HTTP APIs
D. Monitor pattern

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Fan-out/fan-in pattern starts multiple activity functions in parallel using Task.WhenAll (C#) or context.task_all (Python). The orchestrator waits for all tasks to complete and aggregates results. Function chaining (A) is sequential. Async HTTP APIs (C) are for long-running operations with polling. Monitor (D) is for recurring processes.

**Key Concept:** [Fan-out/Fan-in](https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-cloud-backup)
</details>

---

## Domain 2: Develop for Azure Storage (Questions 12-18)

### Question 12
**Scenario:** A developer needs to upload a 5GB video file to Azure Blob Storage from a web application. The upload should resume if interrupted and show progress. What upload method should they use?

A. PutBlob API with single request
B. Block blob upload with staged blocks and commit
C. Append blob
D. Page blob

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Block blobs support uploading in blocks (stages) that can be committed together. If upload is interrupted, only uncommitted blocks need retry. Progress can be tracked per block. PutBlob (A) fails for files > 256MB in single request. Append blobs (C) are for log-type data. Page blobs (D) are for random read/write like VHDs.

**Key Concept:** [Block Blob Upload](https://learn.microsoft.com/en-us/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs)
</details>

### Question 13
**Scenario:** A developer needs to implement optimistic concurrency when updating blobs. Multiple clients may update the same blob simultaneously, and updates should fail if the blob changed since read. What should they use?

A. Lease the blob before update
B. Use ETag and If-Match header in update request
C. Lock the storage account
D. Use Blob versioning

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** ETag represents the blob version. If-Match header with the ETag from read causes update to fail with 412 (Precondition Failed) if blob changed. This implements optimistic concurrency without locking. Lease (A) is pessimistic locking. Storage account lock (C) doesn't exist for this. Versioning (D) keeps history but doesn't prevent concurrent updates.

**Key Concept:** [Optimistic Concurrency](https://learn.microsoft.com/en-us/azure/storage/blobs/concurrency-manage)
</details>

### Question 14
**Scenario:** A developer needs to query data stored in Azure Cosmos DB. The application requires low latency reads from multiple regions and automatic failover. Reads should go to the nearest region. What consistency level and configuration should they use?

A. Strong consistency with single region
B. Session consistency with multi-region writes and preferred locations
C. Eventual consistency only
D. Bounded staleness with manual failover

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Multi-region writes with preferred locations directs reads to the nearest region. Session consistency provides read-your-writes guarantee within a session (good balance of consistency and latency). Automatic failover handles region outages. Strong consistency (A) requires single region. Eventual (C) may return stale data. Manual failover (D) increases RTO.

**Key Concept:** [Cosmos DB Multi-region](https://learn.microsoft.com/en-us/azure/cosmos-db/distribute-data-globally)
</details>

### Question 15
**Scenario:** A developer is designing a Cosmos DB container for an e-commerce application. The main access patterns are: get orders by customer ID, get orders by order date range, and get order by order ID. What partition key should they choose?

A. Order ID
B. Customer ID
C. Order Date
D. Composite key of Customer ID and Order Date

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Customer ID groups related orders together for efficient customer-based queries. Single-item lookup by Order ID works with any partition key (cross-partition query). Date range queries across customers would be cross-partition but are typically less frequent. Order ID (A) would make every query cross-partition except single-item. Date (C) creates hot partitions. Composite (D) complicates customer queries.

**Key Concept:** [Partition Key Selection](https://learn.microsoft.com/en-us/azure/cosmos-db/partitioning-overview)
</details>

### Question 16
**Scenario:** A developer needs to store and retrieve user session data with TTL (time-to-live). Sessions should automatically expire after 30 minutes of inactivity. What Azure service should they use?

A. Azure Table Storage with manual cleanup
B. Azure Cache for Redis with key expiration
C. Azure Blob Storage with lifecycle management
D. Azure SQL with scheduled job

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Redis supports key expiration (TTL) natively with sliding expiration on access. Session data is accessed frequently and benefits from in-memory speed. Built-in TTL automatically removes expired sessions. Table Storage (A) requires manual cleanup. Blob lifecycle (C) minimum is days, not minutes. SQL job (D) adds complexity.

**Key Concept:** [Redis Key Expiration](https://learn.microsoft.com/en-us/azure/azure-cache-for-redis/cache-best-practices-development)
</details>

### Question 17
**Scenario:** A developer needs to perform a batch operation in Cosmos DB that updates 100 items atomically. All updates must succeed or all must fail. What should they use?

A. Individual update operations in a loop
B. Transactional batch with the same partition key
C. Stored procedure
D. Bulk executor

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Transactional batch operations in Cosmos DB are atomic within a single partition key. All operations succeed or all fail with no partial updates. Individual updates (A) aren't atomic. Stored procedures (C) also require same partition but are more complex. Bulk executor (D) is for high-throughput, not transactions.

**Key Concept:** [Transactional Batch](https://learn.microsoft.com/en-us/azure/cosmos-db/nosql/transactional-batch)
</details>

### Question 18
**Scenario:** A developer needs to generate a URL that allows a third party to upload a file directly to Blob Storage for the next hour. The third party should only be able to upload, not read or delete. What should they generate?

A. Shared Access Signature (SAS) with create permission and 1-hour expiry
B. Account key URL
C. Public container URL
D. Azure AD token

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** SAS tokens provide time-limited, permission-scoped access. Create (c) permission allows upload but not read or delete. 1-hour expiry limits the time window. Account key (B) gives full access. Public container (C) allows read, not upload. Azure AD (D) requires third-party Azure AD integration.

**Key Concept:** [Blob SAS](https://learn.microsoft.com/en-us/azure/storage/common/storage-sas-overview)
</details>

---

## Domain 3: Implement Azure Security (Questions 19-27)

### Question 19
**Scenario:** A developer needs to access Azure Key Vault secrets from an Azure Function without storing credentials. The Function runs on a Consumption plan. What authentication method should they use?

A. Store Key Vault access key in App Settings
B. System-assigned managed identity with Key Vault access policy
C. User-assigned managed identity only
D. Certificate-based authentication

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** System-assigned managed identity is automatically created and managed by Azure. Access policy (or RBAC) grants the identity permission to read secrets. No credentials to manage. Consumption plan supports managed identity. Storing keys (A) defeats the purpose. User-assigned (C) works but system-assigned is simpler for single-resource scenarios.

**Key Concept:** [Managed Identity](https://learn.microsoft.com/en-us/azure/app-service/overview-managed-identity)
</details>

### Question 20
**Scenario:** A developer is building a web API that authenticates users with Azure AD. The API needs to validate the access token and extract user claims. What should they implement?

A. Custom token validation code
B. Microsoft.Identity.Web library with AddMicrosoftIdentityWebApi
C. Manual JWT parsing
D. API Management validation policy

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Microsoft.Identity.Web simplifies Azure AD integration with a single line of configuration. It handles token validation, issuer verification, audience checking, and claims extraction. Custom validation (A) or manual parsing (C) risks security vulnerabilities. API Management (D) is for gateway-level validation, not application-level.

**Key Concept:** [Microsoft.Identity.Web](https://learn.microsoft.com/en-us/azure/active-directory/develop/microsoft-identity-web)
</details>

### Question 21
**Scenario:** A developer needs to store an API key that the application uses to call an external service. The key should be encrypted, audited, and rotatable without redeploying the application. Where should they store it?

A. Application configuration file
B. Azure Key Vault secret with application reference
C. Environment variable
D. Database table

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Key Vault secrets are encrypted at rest and in transit. Access is audited via diagnostic logs. Key rotation updates the secret without application redeployment if using Key Vault references. App Service/Functions can reference Key Vault secrets in configuration. Config files (A) and environment variables (C) aren't encrypted or audited. Database (D) lacks specialized secret management.

**Key Concept:** [Key Vault References](https://learn.microsoft.com/en-us/azure/app-service/app-service-key-vault-references)
</details>

### Question 22
**Scenario:** A developer needs to implement the On-Behalf-Of (OBO) flow where a web API calls a downstream API using the user's identity. The original access token was obtained by a SPA. What should they implement?

A. Use the original access token directly
B. Exchange the incoming token for a new token using MSAL's AcquireTokenOnBehalfOf
C. Request a new token with client credentials
D. Pass user credentials to downstream API

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** OBO flow exchanges the user's access token for a new token scoped for the downstream API. MSAL handles this exchange with Azure AD. The downstream API receives a token representing the original user. Using original token (A) may have wrong audience. Client credentials (C) loses user context. Passing credentials (D) is insecure.

**Key Concept:** [On-Behalf-Of Flow](https://learn.microsoft.com/en-us/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow)
</details>

### Question 23
**Scenario:** A developer needs to configure Azure App Service to only accept HTTPS traffic and enforce a minimum TLS version of 1.2. What settings should they configure?

A. Configure in application code only
B. Enable HTTPS Only setting and set minimum TLS version in App Service configuration
C. Use Application Gateway for TLS termination
D. Configure web.config only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** App Service has built-in settings for HTTPS Only (redirects HTTP to HTTPS) and minimum TLS version. These are platform-level enforcement independent of application code. Application code (A) can be bypassed. Application Gateway (C) is additional complexity. Web.config (D) is application-level for IIS, not platform-level.

**Key Concept:** [App Service TLS](https://learn.microsoft.com/en-us/azure/app-service/configure-ssl-bindings)
</details>

### Question 24
**Scenario:** A developer is implementing certificate-based authentication for a service-to-service scenario. The calling service needs to authenticate to an Azure AD-protected API using a certificate. What authentication flow should they use?

A. Authorization code flow
B. Client credentials flow with certificate
C. Implicit flow
D. Device code flow

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Client credentials flow with certificate is designed for service-to-service authentication where there's no user. The certificate proves the identity of the calling application. Authorization code (A) and implicit (C) are user-interactive flows. Device code (D) is for devices without browsers.

**Key Concept:** [Client Credentials with Certificate](https://learn.microsoft.com/en-us/azure/active-directory/develop/msal-net-client-assertions)
</details>

### Question 25
**Scenario:** A developer needs to call Microsoft Graph API from an Azure Function to read user profiles. The function runs on a schedule (no user context). What permissions and authentication should they use?

A. Delegated permissions with user sign-in
B. Application permissions with client credentials flow using managed identity
C. API key
D. Basic authentication

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Application permissions allow accessing Graph API without user context (daemon/service scenario). Client credentials flow authenticates the application. Managed identity simplifies credential management. Admin consent is required for application permissions. Delegated permissions (A) require user context. API key (C) and basic auth (D) aren't supported by Graph.

**Key Concept:** [Graph Application Permissions](https://learn.microsoft.com/en-us/graph/auth/auth-concepts)
</details>

### Question 26
**Scenario:** A developer needs to encrypt sensitive data before storing it in a database. The encryption key should be managed by Azure Key Vault. What approach should they use?

A. Always Encrypted with Key Vault
B. Store encrypted data with TDE
C. Manual encryption with downloaded key
D. Database-level encryption only

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Always Encrypted encrypts data at the client before sending to database. Column master key (CMK) stored in Key Vault protects column encryption keys. Data remains encrypted in the database. TDE (B) is transparent and doesn't protect data in memory or transit. Manual encryption (C) exposes key. Database-level only (D) exposes data to DBAs.

**Key Concept:** [Always Encrypted](https://learn.microsoft.com/en-us/sql/relational-databases/security/encryption/always-encrypted-database-engine)
</details>

### Question 27
**Scenario:** A developer needs to implement API authentication that supports multiple identity providers (Azure AD, social providers, and local accounts) for a web application. What should they use?

A. Implement separate authentication for each provider
B. Azure AD B2C with user flows or custom policies
C. Azure AD only
D. Custom authentication system

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure AD B2C supports federation with multiple identity providers (social, enterprise, local accounts) through a unified interface. User flows provide common scenarios without code. Custom policies enable advanced scenarios. Separate implementations (A) are complex. Azure AD only (C) doesn't support social providers natively. Custom auth (D) is insecure.

**Key Concept:** [Azure AD B2C Identity Providers](https://learn.microsoft.com/en-us/azure/active-directory-b2c/add-identity-provider)
</details>

---

## Domain 4: Monitor, Troubleshoot, and Optimize (Questions 28-34)

### Question 28
**Scenario:** A developer needs to track custom business metrics (e.g., orders processed, revenue) in Application Insights. These metrics should be available for alerting and dashboards. What should they implement?

A. Write to log files only
B. TrackMetric or GetMetric API in application code
C. Use only built-in metrics
D. Export to external system

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** TrackMetric sends custom metrics to Application Insights. GetMetric provides local aggregation for high-frequency metrics. Custom metrics appear in Metrics Explorer, can trigger alerts, and be used in dashboards. Log files (A) aren't metrics. Built-in (C) doesn't include business metrics. Export (D) adds latency.

**Key Concept:** [Custom Metrics](https://learn.microsoft.com/en-us/azure/azure-monitor/app/api-custom-events-metrics)
</details>

### Question 29
**Scenario:** A developer's web application has intermittent slow responses. They need to identify which dependency call is causing the latency. What Application Insights feature should they use?

A. Availability tests
B. Application Map with dependency tracking
C. Usage analytics
D. Smart detection only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Application Map visualizes dependencies and their performance. Dependency tracking shows average duration and failure rates for external calls (databases, APIs, etc.). Clicking on slow dependencies shows details. Availability tests (A) check endpoint availability. Usage analytics (C) tracks user behavior. Smart detection (D) is automatic alerting.

**Key Concept:** [Application Map](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-map)
</details>

### Question 30
**Scenario:** A developer needs to implement distributed tracing across multiple microservices to track a request from start to finish. What should they configure?

A. Separate Application Insights resources for each service
B. Shared Application Insights with W3C Trace Context correlation
C. Manual logging with timestamps
D. Separate log files per service

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** W3C Trace Context provides standard correlation headers (traceparent, tracestate) across services. Application Insights SDK automatically propagates correlation IDs. All services using the same connection string correlate traces in End-to-End Transaction view. Separate resources (A) lose correlation. Manual logging (C, D) requires custom correlation logic.

**Key Concept:** [Distributed Tracing](https://learn.microsoft.com/en-us/azure/azure-monitor/app/distributed-tracing)
</details>

### Question 31
**Scenario:** A developer needs to configure alerts when their Azure Function experiences more than 10 failures in 5 minutes. They want to receive email notifications. What should they configure?

A. Application Insights availability alert
B. Azure Monitor alert rule with metric condition on Function errors
C. Log alert only
D. Manual monitoring

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Monitor alert rules can trigger on Function-specific metrics (Function Execution Count with ResultType=Failed). Condition: greater than 10 in 5-minute window. Action group sends email notification. Availability alerts (A) are for endpoint checks. Log alerts (C) work but metric alerts are more efficient for this scenario.

**Key Concept:** [Function Monitoring](https://learn.microsoft.com/en-us/azure/azure-functions/functions-monitoring)
</details>

### Question 32
**Scenario:** A developer needs to optimize a Cosmos DB application that is experiencing high RU charges. Many queries return entire documents but only need a few fields. What optimization should they implement?

A. Increase provisioned throughput
B. Use projection in queries to return only needed fields
C. Index all fields
D. Disable indexing

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Projection (SELECT c.field1, c.field2) reduces response payload size, lowering RU cost. Reading entire documents when only fields are needed wastes RUs. Increasing throughput (A) adds cost without solving efficiency. More indexes (C) may increase write RU. Disabling indexing (D) increases query RU.

**Key Concept:** [Query Optimization](https://learn.microsoft.com/en-us/azure/cosmos-db/nosql/query/pagination)
</details>

### Question 33
**Scenario:** A developer notices their Azure Function has memory issues. They need to analyze memory usage patterns over time. What tool should they use?

A. Azure Portal metrics only
B. Application Insights Profiler and live metrics
C. Visual Studio debugger in production
D. Print statements

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Application Insights Profiler captures performance traces including memory allocation. Live Metrics shows real-time memory usage. Together they identify memory patterns and leaks. Portal metrics (A) show high-level memory but not allocation details. VS debugger (C) shouldn't run in production. Print statements (D) don't show memory details.

**Key Concept:** [Application Insights Profiler](https://learn.microsoft.com/en-us/azure/azure-monitor/profiler/profiler-overview)
</details>

### Question 34
**Scenario:** A developer needs to implement caching for an API that returns user profile data. The cache should invalidate when user data changes. What caching pattern should they use?

A. Cache-aside with explicit invalidation on data changes
B. Read-through cache only
C. Write-through cache only
D. Time-based expiration only

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Cache-aside pattern: application checks cache first, loads from database on miss, caches result. On data changes, application invalidates (deletes) cache entry. Next read fetches fresh data. Read-through (B) doesn't handle invalidation. Write-through (C) keeps cache updated but adds write latency. Time-based only (D) may serve stale data.

**Key Concept:** [Caching Patterns](https://learn.microsoft.com/en-us/azure/architecture/patterns/cache-aside)
</details>

---

## Domain 5: Connect to and Consume Azure Services (Questions 35-40)

### Question 35
**Scenario:** A developer needs to implement reliable messaging between two services. Messages should be processed in order and duplicates should be prevented. What Azure service should they use?

A. Azure Queue Storage
B. Azure Service Bus with FIFO queue (sessions)
C. Azure Event Grid
D. Azure Event Hubs

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Service Bus queues with sessions provide FIFO ordering within a session. Duplicate detection prevents duplicate messages within a configurable window. Service Bus provides exactly-once processing semantics. Queue Storage (A) doesn't guarantee order. Event Grid (C) is for event routing. Event Hubs (D) is for streaming, not message queuing.

**Key Concept:** [Service Bus Sessions](https://learn.microsoft.com/en-us/azure/service-bus-messaging/message-sessions)
</details>

### Question 36
**Scenario:** A developer needs to implement a pub/sub pattern where a single event triggers multiple independent subscribers. Each subscriber should receive the event. What Azure service and pattern should they use?

A. Service Bus Queue with competing consumers
B. Service Bus Topic with multiple subscriptions
C. Event Grid with single endpoint
D. Storage Queue

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Service Bus Topics support pub/sub with multiple subscriptions. Each subscription receives a copy of every message (or filtered messages). Subscribers are independent. Queue (A) is competing consumers (one receives each message). Event Grid (C) with single endpoint is one subscriber. Storage Queue (D) doesn't support pub/sub.

**Key Concept:** [Service Bus Topics](https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-queues-topics-subscriptions)
</details>

### Question 37
**Scenario:** A developer needs to handle Azure resource events (e.g., blob created, VM state changed) and trigger Azure Functions. Events should be delivered with at-least-once guarantee. What should they use?

A. Polling for changes
B. Azure Event Grid with Function subscriber
C. Azure Monitor alerts
D. Azure Automation webhooks

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Event Grid provides reactive, event-driven triggers for Azure resource events. Native integration with Functions as event handlers. At-least-once delivery with retry. Many Azure services publish events to Event Grid natively. Polling (A) is inefficient. Monitor alerts (C) are for metrics/logs. Automation webhooks (D) are for runbook triggers.

**Key Concept:** [Event Grid](https://learn.microsoft.com/en-us/azure/event-grid/overview)
</details>

### Question 38
**Scenario:** A developer is building a web API that calls a backend service. They need to implement retry logic, circuit breaker, and timeout policies for the HTTP calls. What library should they use?

A. Manual try-catch with Thread.Sleep
B. Polly library with IHttpClientFactory
C. HttpClient without any policies
D. WebClient class

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Polly is a resilience library supporting retry, circuit breaker, timeout, fallback, and more. IHttpClientFactory integrates Polly policies with HttpClient lifecycle management. Policies are configured declaratively. Manual retry (A) is error-prone. Plain HttpClient (C) has no resilience. WebClient (D) is legacy.

**Key Concept:** [HttpClientFactory with Polly](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/implement-http-call-retries-exponential-backoff-polly)
</details>

### Question 39
**Scenario:** A developer needs to expose an Azure Function as an API with rate limiting, authentication, and request/response transformation. What should they use?

A. Function authorization keys only
B. Azure API Management in front of the Function
C. Custom middleware in the Function
D. Application Gateway

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** API Management provides rate limiting (quotas, throttling), multiple authentication options (keys, OAuth, certificates), and policies for transformation. Functions are configured as APIM backends. Function keys (A) provide basic auth but not rate limiting. Custom middleware (C) requires building these features. Application Gateway (D) is for routing/WAF, not API management.

**Key Concept:** [API Management](https://learn.microsoft.com/en-us/azure/api-management/api-management-key-concepts)
</details>

### Question 40
**Scenario:** A developer needs to implement feature flags that can be changed without redeploying the application. The flags should support gradual rollouts (percentage-based) and targeting specific users.

A. Application configuration files
B. Azure App Configuration with feature management and targeting filters
C. Environment variables
D. Database table with feature flags

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** App Configuration feature management supports feature flags with targeting filters (user ID, percentage, custom filters). Changes are reflected without deployment. SDK integration provides easy feature checks. Config files (A) and environment variables (C) require deployment. Database (D) requires building targeting logic.

**Key Concept:** [App Configuration Feature Flags](https://learn.microsoft.com/en-us/azure/azure-app-configuration/concept-feature-management)
</details>

---

## Answer Key

| Question | Answer | Domain |
|----------|--------|--------|
| 1 | B | Compute |
| 2 | B | Compute |
| 3 | B | Compute |
| 4 | B | Compute |
| 5 | B | Compute |
| 6 | B | Compute |
| 7 | B | Compute |
| 8 | C | Compute |
| 9 | B | Compute |
| 10 | B | Compute |
| 11 | B | Compute |
| 12 | B | Storage |
| 13 | B | Storage |
| 14 | B | Storage |
| 15 | B | Storage |
| 16 | B | Storage |
| 17 | B | Storage |
| 18 | A | Storage |
| 19 | B | Security |
| 20 | B | Security |
| 21 | B | Security |
| 22 | B | Security |
| 23 | B | Security |
| 24 | B | Security |
| 25 | B | Security |
| 26 | A | Security |
| 27 | B | Security |
| 28 | B | Monitor/Optimize |
| 29 | B | Monitor/Optimize |
| 30 | B | Monitor/Optimize |
| 31 | B | Monitor/Optimize |
| 32 | B | Monitor/Optimize |
| 33 | B | Monitor/Optimize |
| 34 | A | Monitor/Optimize |
| 35 | B | Azure Services |
| 36 | B | Azure Services |
| 37 | B | Azure Services |
| 38 | B | Azure Services |
| 39 | B | Azure Services |
| 40 | B | Azure Services |
