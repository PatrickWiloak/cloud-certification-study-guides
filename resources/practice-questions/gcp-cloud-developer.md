# GCP Professional Cloud Developer Practice Questions

40 scenario-based questions weighted by exam domain.

## Exam Domain Breakdown
| Domain | Weight | Questions |
|--------|--------|-----------|
| Designing highly scalable, available, and reliable cloud-native applications | 26% | 10 |
| Building and testing applications | 20% | 8 |
| Deploying applications | 18% | 7 |
| Integrating Google Cloud services | 22% | 9 |
| Managing deployed applications | 14% | 6 |

---

## Domain 1: Designing Highly Scalable, Available, and Reliable Cloud-Native Applications (Questions 1-10)

### Question 1
**Scenario:** A startup is building a social media application expecting rapid growth from 10,000 to 10 million users. They need a database solution that can handle high read/write throughput, scale horizontally without downtime, and support complex queries for user feeds. Cost optimization is important in early stages. Which database solution is BEST?

A. Cloud SQL with read replicas
B. Cloud Spanner with regional configuration
C. Firestore in Native mode with composite indexes
D. Bigtable for all data storage

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Firestore in Native mode provides automatic horizontal scaling, real-time synchronization (useful for social feeds), offline support for mobile apps, and a generous free tier for cost optimization in early stages. Composite indexes support complex queries for feeds. Cloud SQL (A) requires manual scaling and has replication lag. Spanner (B) is powerful but expensive for startups. Bigtable (D) is optimized for time-series, not social app queries.

**Key Concept:** [Firestore Overview](https://cloud.google.com/firestore/docs/overview)
</details>

### Question 2
**Scenario:** An e-commerce application needs to process orders with guaranteed exactly-once delivery. Orders must be processed in the sequence received from each customer, but orders from different customers can be processed in parallel. What messaging architecture supports this?

A. Pub/Sub with pull subscriptions and manual acknowledgment
B. Pub/Sub with ordering keys set to customer ID and exactly-once delivery enabled
C. Cloud Tasks with named tasks per order
D. Direct HTTP calls with retry logic

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Pub/Sub ordering keys ensure messages with the same key (customer ID) are delivered in order. Exactly-once delivery (through Dataflow or subscriber logic with acknowledgment) ensures no duplicate processing. Different customers have different keys, enabling parallel processing. Pull subscriptions alone (A) don't guarantee ordering. Cloud Tasks (C) is for task queuing, not event streaming. Direct HTTP (D) doesn't guarantee delivery or ordering.

**Key Concept:** [Pub/Sub Ordering](https://cloud.google.com/pubsub/docs/ordering)
</details>

### Question 3
**Scenario:** A gaming company needs to store player session data that must persist across server restarts, be accessible globally with low latency, and automatically expire after 24 hours of inactivity. What storage solution meets these requirements?

A. Cloud Storage with lifecycle policies
B. Memorystore for Redis with TTL and replication
C. Firestore with TTL policies
D. Cloud SQL with scheduled cleanup jobs

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Firestore provides global distribution through multi-region instances with strong consistency, automatic TTL policies for document expiration, and persistence across server restarts. It's designed for real-time data that needs global access. Memorystore (B) is regional and primarily for caching. Cloud Storage (A) has high latency for session data. Cloud SQL (D) requires manual cleanup and doesn't scale globally as easily.

**Key Concept:** [Firestore TTL Policies](https://cloud.google.com/firestore/docs/ttl)
</details>

### Question 4
**Scenario:** A financial application needs to ensure that a money transfer operation (debit from one account, credit to another) either completes fully or not at all, even across service boundaries in a microservices architecture. How should this be implemented?

A. Use distributed transactions with two-phase commit across services
B. Implement the Saga pattern with compensating transactions using Cloud Tasks or Workflows
C. Use a single Cloud SQL database for all services to enable transactions
D. Implement optimistic locking with retry logic

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** The Saga pattern handles distributed transactions by breaking them into local transactions with compensating actions for rollback. Cloud Workflows or Cloud Tasks can orchestrate the saga, ensuring that if the credit fails, the debit is reversed. This maintains loose coupling in microservices. Two-phase commit (A) creates tight coupling and doesn't scale. Single database (C) defeats microservices benefits. Optimistic locking (D) doesn't handle cross-service consistency.

**Key Concept:** [Saga Pattern](https://cloud.google.com/architecture/managing-data-consistency-microservices-two-phase-commit)
</details>

### Question 5
**Scenario:** An IoT platform receives millions of sensor readings per second from devices worldwide. Data must be ingested, processed for anomalies in real-time, stored for analysis, and made available for dashboards with sub-second latency. What architecture supports these requirements?

A. Devices → Cloud Functions → Cloud SQL → Data Studio
B. Devices → Pub/Sub → Dataflow (streaming) → BigQuery + Bigtable → Looker
C. Devices → IoT Core → Cloud Storage → BigQuery
D. Devices → Compute Engine → Cloud SQL → custom dashboard

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Pub/Sub handles high-throughput ingestion with global endpoints. Dataflow streaming processes data in real-time for anomaly detection. BigQuery stores data for analytical queries. Bigtable provides low-latency reads for dashboard queries requiring sub-second response. Looker connects to both. Cloud Functions (A) don't scale for millions per second. Batch to Cloud Storage (C) isn't real-time. Compute Engine (D) requires manual scaling.

**Key Concept:** [IoT Reference Architecture](https://cloud.google.com/architecture/connected-devices)
</details>

### Question 6
**Scenario:** A media company's video processing pipeline occasionally fails midway through processing. They need to ensure that failed jobs can be retried from where they left off rather than restarting from the beginning. What design pattern addresses this?

A. Idempotent operations with checkpointing
B. Complete transaction rollback on failure
C. Synchronous processing with immediate retry
D. Processing without state management

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Idempotent operations can be safely retried without side effects. Checkpointing saves progress at intervals (e.g., in Cloud Storage or Firestore), allowing processing to resume from the last checkpoint on retry. This is essential for long-running jobs. Rollback (B) loses all progress. Synchronous retry (C) still restarts from beginning. Stateless processing (D) can't resume from failure points.

**Key Concept:** [Designing for Reliability](https://cloud.google.com/architecture/scalable-and-resilient-apps)
</details>

### Question 7
**Scenario:** A ride-sharing application needs to match riders with nearby drivers in real-time. The system must handle 100,000 concurrent location updates per minute and return matches within 100ms. What architecture supports this?

A. Store locations in Cloud SQL with spatial queries
B. Use Memorystore for Redis with geospatial commands (GEOADD, GEORADIUS)
C. Store locations in BigQuery and use UDFs for distance calculation
D. Use Cloud Functions with in-memory caching

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Redis geospatial commands are optimized for this exact use case - GEOADD stores locations, GEORADIUS finds nearby points with sub-millisecond latency. Memorystore for Redis handles high throughput with automatic scaling. Cloud SQL (A) spatial queries have higher latency. BigQuery (C) is for analytics, not real-time queries. Cloud Functions (D) don't share state between instances.

**Key Concept:** [Memorystore for Redis](https://cloud.google.com/memorystore/docs/redis/redis-overview)
</details>

### Question 8
**Scenario:** An application needs to call multiple independent external APIs to assemble a response. Each API has variable latency (50-500ms). The application should return as quickly as possible while being resilient to individual API failures. How should this be designed?

A. Call APIs sequentially and aggregate results
B. Call APIs in parallel using async/await with timeouts and fallback values for failures
C. Use a single API gateway that aggregates all external APIs
D. Cache all API responses indefinitely

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Parallel calls ensure total latency is the maximum of individual calls, not the sum. Timeouts prevent slow APIs from blocking the response. Fallback values (e.g., cached data, defaults) handle failures gracefully, implementing the circuit breaker pattern. Sequential calls (A) have cumulative latency. API gateway (C) still needs parallel calling logic. Indefinite caching (D) serves stale data.

**Key Concept:** [Resilient Applications](https://cloud.google.com/architecture/scalable-and-resilient-apps#use_circuit_breakers)
</details>

### Question 9
**Scenario:** A batch processing system runs nightly jobs that vary dramatically in resource needs - some jobs need 2 CPUs for 10 minutes, others need 64 CPUs for 2 hours. The company wants to minimize costs while ensuring jobs complete on time. What compute solution is BEST?

A. Dedicated Compute Engine VMs sized for the largest job
B. Cloud Run jobs with appropriate CPU/memory configuration per job
C. GKE Autopilot with job-specific resource requests
D. Dataflow with autoscaling for batch jobs

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Run jobs allow specifying CPU and memory per job type, scale to zero when not running, and bill only for actual execution time. Different job configurations can have different resource allocations. This minimizes costs for variable workloads. Dedicated VMs (A) waste resources during light jobs. GKE Autopilot (C) works but has more overhead than Cloud Run for simple batch jobs. Dataflow (D) is for data processing pipelines, not general batch jobs.

**Key Concept:** [Cloud Run Jobs](https://cloud.google.com/run/docs/create-jobs)
</details>

### Question 10
**Scenario:** A content platform serves static assets (images, videos) to users globally. They need to minimize latency for users worldwide, handle traffic spikes during viral content, and reduce origin server load. What configuration provides this?

A. Cloud Storage with multi-regional bucket
B. Cloud Storage with Cloud CDN enabled
C. Compute Engine instances in each region serving content
D. Cloud Storage with signed URLs only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud CDN caches content at Google's edge locations worldwide, providing low latency to users regardless of location. It handles traffic spikes through edge caching without impacting origin. Cloud Storage serves as the origin. Multi-regional buckets (A) provide redundancy but not edge caching. Regional Compute Engine (C) doesn't provide global edge presence. Signed URLs (D) provide security but not performance.

**Key Concept:** [Cloud CDN](https://cloud.google.com/cdn/docs/overview)
</details>

---

## Domain 2: Building and Testing Applications (Questions 11-18)

### Question 11
**Scenario:** A development team wants to run integration tests that require a Cloud SQL database. Tests should be isolated, run in CI/CD, and not affect production data. The database schema is complex with many tables. What is the BEST approach?

A. Use the production database with test data prefixes
B. Create a Cloud SQL instance per test run and delete after
C. Use Cloud SQL Auth Proxy with a dedicated test instance and transaction rollback after each test
D. Mock all database interactions in tests

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** A dedicated test instance ensures isolation from production. Cloud SQL Auth Proxy provides secure access from CI/CD. Transaction rollback after each test (or test suites) ensures test isolation without recreating the database. This balances realism with practicality. Production database (A) risks data corruption. Instance per run (B) is slow and expensive. Mocking (D) doesn't test actual database behavior.

**Key Concept:** [Cloud SQL Auth Proxy](https://cloud.google.com/sql/docs/mysql/sql-proxy)
</details>

### Question 12
**Scenario:** A team needs to test their Cloud Functions locally before deployment. The functions interact with Pub/Sub, Firestore, and Cloud Storage. How can they set up a local development environment that mimics production?

A. Deploy to a development project and test there
B. Use the Functions Framework locally with Firebase Emulator Suite for Firestore and local Pub/Sub emulator
C. Write unit tests that mock all GCP services
D. Use production services with test data

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** The Functions Framework allows running Cloud Functions locally. Firebase Emulator Suite provides local emulation of Firestore and other Firebase services. Pub/Sub emulator simulates Pub/Sub locally. This enables rapid iteration without cloud costs or affecting other developers. Deploying to dev (A) is slower. Pure mocks (C) don't catch integration issues. Production services (D) risk data issues and incur costs.

**Key Concept:** [Functions Framework](https://cloud.google.com/functions/docs/functions-framework)
</details>

### Question 13
**Scenario:** A company wants to ensure their container images don't contain known vulnerabilities before deployment to GKE. They also want to block deployment of images that haven't been scanned. How should this be implemented?

A. Manual security review before deployment
B. Enable Container Analysis for vulnerability scanning and Binary Authorization with attestation requirements
C. Use only official base images
D. Scan images after deployment and fix issues

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Container Analysis automatically scans images in Artifact Registry for vulnerabilities. Binary Authorization can require attestations (including vulnerability scan results) before images can be deployed to GKE, blocking non-compliant images. This creates a secure software supply chain. Manual review (A) doesn't scale. Official images (C) still need scanning. Post-deployment scanning (D) is reactive, not preventive.

**Key Concept:** [Binary Authorization](https://cloud.google.com/binary-authorization/docs/overview)
</details>

### Question 14
**Scenario:** A development team needs to manage application configuration that varies between development, staging, and production environments. Configuration includes feature flags, API endpoints, and database connection strings. What is the BEST approach?

A. Hard-code configuration with if/else for each environment
B. Use environment variables injected at deployment time, with secrets in Secret Manager
C. Store all configuration in the application repository
D. Use a single configuration for all environments

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Environment variables allow the same application code to run in different environments with different configuration. Secret Manager securely stores sensitive values like database credentials, accessible via APIs. This follows the 12-factor app methodology. Hard-coding (A) requires code changes for config changes. Repository storage (C) risks exposing secrets. Single config (D) doesn't support environment-specific needs.

**Key Concept:** [Secret Manager](https://cloud.google.com/secret-manager/docs/overview)
</details>

### Question 15
**Scenario:** A team is developing a machine learning model training pipeline. They need to track experiments, compare model versions, and reproduce results. The pipeline uses Vertex AI for training. How should they manage this?

A. Store experiment results in spreadsheets
B. Use Vertex AI Experiments for tracking with Vertex AI Pipelines for reproducibility
C. Save model files to Cloud Storage with manual versioning
D. Use Git branches for each experiment

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Vertex AI Experiments tracks metrics, parameters, and artifacts for each training run, enabling comparison. Vertex AI Pipelines captures the entire workflow as code, ensuring reproducibility. Metadata is automatically logged. Spreadsheets (A) are error-prone and don't capture artifacts. Manual versioning (C) doesn't track parameters and metrics. Git branches (D) don't track runtime metrics.

**Key Concept:** [Vertex AI Experiments](https://cloud.google.com/vertex-ai/docs/experiments/intro-vertex-ai-experiments)
</details>

### Question 16
**Scenario:** A microservices application has intermittent failures that are difficult to reproduce. The team needs to understand the flow of requests across services and identify where latency or errors occur. What observability solution helps with this?

A. Application logs in Cloud Logging
B. Cloud Trace for distributed tracing combined with Cloud Profiler for performance
C. Cloud Monitoring metrics only
D. Custom logging in each service

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Trace provides distributed tracing, showing request flow across services with latency breakdowns. This reveals which service causes delays or errors. Cloud Profiler identifies code-level performance issues. Together they provide comprehensive observability. Logs (A, D) show individual events but not request flow. Metrics (C) show aggregates but not individual request paths.

**Key Concept:** [Cloud Trace](https://cloud.google.com/trace/docs/overview)
</details>

### Question 17
**Scenario:** A developer needs to test error handling code paths, including simulating network failures, service unavailability, and timeout scenarios. The application uses gRPC for service communication. How can they reliably test these scenarios?

A. Wait for random production failures
B. Use fault injection with Istio service mesh or test doubles that simulate failures
C. Disable network connectivity during tests
D. Hope error handling code is correct

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Istio service mesh provides fault injection capabilities - injecting delays, HTTP errors, and gRPC failures between services. Test doubles (mocks/stubs) can also simulate failures. This enables systematic testing of error paths. Waiting for production failures (A) is unreliable and risky. Disabling network (C) is blunt and doesn't test specific failure modes. Hope (D) isn't a testing strategy.

**Key Concept:** [Istio Fault Injection](https://istio.io/latest/docs/tasks/traffic-management/fault-injection/)
</details>

### Question 18
**Scenario:** A team wants to automate code review checks including linting, security scanning, and test coverage requirements. Pull requests should not be mergeable until all checks pass. They use GitHub for source control. How should this be implemented?

A. Manual code review only
B. Cloud Build triggers on pull requests running checks, with GitHub branch protection requiring status checks
C. Run checks only after merging
D. Trust developers to run checks locally

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Build can trigger on GitHub pull requests, running linting, security scans (e.g., with tools like Snyk or Trivy), and tests. GitHub branch protection rules can require these status checks to pass before merging. This automates quality gates. Manual review (A) misses automated checks. Post-merge checks (C) are too late. Trusting local runs (D) isn't enforceable.

**Key Concept:** [Cloud Build GitHub Integration](https://cloud.google.com/build/docs/automating-builds/github/build-repos-from-github)
</details>

---

## Domain 3: Deploying Applications (Questions 19-25)

### Question 19
**Scenario:** A web application needs to be deployed with zero downtime. The team wants to gradually shift traffic to new versions and automatically roll back if error rates increase. The application runs on GKE. What deployment strategy supports this?

A. Recreate deployment - stop old, start new
B. Canary deployment with GKE Gateway and Cloud Deploy
C. Blue-green deployment with manual DNS switching
D. Deploy directly to production nodes

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Canary deployments route a small percentage of traffic to the new version initially. GKE Gateway (or Istio) enables traffic splitting. Cloud Deploy can automate promotion with verification, including automatic rollback based on Cloud Monitoring metrics (error rates). Recreate (A) causes downtime. Manual blue-green (C) doesn't have automatic rollback. Direct deployment (D) risks full outage on failure.

**Key Concept:** [Cloud Deploy](https://cloud.google.com/deploy/docs/overview)
</details>

### Question 20
**Scenario:** A company has multiple microservices that need to be deployed together as they have interdependencies. Some services need database migrations to run before deployment. How should this deployment be orchestrated?

A. Deploy services manually in the correct order
B. Use Cloud Deploy with a delivery pipeline defining stages and dependencies
C. Deploy all services simultaneously
D. Use cron jobs to coordinate deployments

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Deploy delivery pipelines can define deployment stages, ordering, and promotion rules. Pre-deploy hooks can run database migrations. Parallel targets can deploy independent services simultaneously while respecting dependencies. Manual deployment (A) is error-prone. Simultaneous deployment (C) ignores dependencies. Cron jobs (D) are timing-based, not dependency-aware.

**Key Concept:** [Cloud Deploy Pipelines](https://cloud.google.com/deploy/docs/deploy-app-hooks)
</details>

### Question 21
**Scenario:** A stateless API application needs to scale from 0 to handle sporadic traffic patterns with unpredictable bursts. Cold start time should be minimal. The application is containerized. Which compute option is BEST?

A. GKE Autopilot with HPA
B. Cloud Run with minimum instances set to 0 and concurrency tuning
C. Compute Engine with managed instance groups
D. Cloud Functions for containers

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Run scales to zero (cost savings during no traffic) and scales up rapidly on demand. Setting minimum instances > 0 eliminates cold starts if needed, but for cost optimization with sporadic traffic, 0 minimum works. Concurrency tuning optimizes instance utilization. GKE Autopilot (A) has higher baseline costs. Compute Engine (C) has slower scaling. Cloud Functions (D) is for event-driven, not HTTP APIs with containers.

**Key Concept:** [Cloud Run Autoscaling](https://cloud.google.com/run/docs/configuring/min-instances)
</details>

### Question 22
**Scenario:** A team needs to deploy infrastructure and applications together in a repeatable way. They want to define both GKE clusters and the applications running on them as code. What approach provides this?

A. Manual deployment through Cloud Console
B. Terraform for infrastructure with Config Connector for Kubernetes resources
C. gcloud commands in shell scripts
D. Separate teams managing infrastructure and applications

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Terraform manages GCP infrastructure as code (VPCs, GKE clusters, etc.). Config Connector allows defining GCP resources as Kubernetes manifests, bridging infrastructure and application deployment in a unified GitOps workflow. This enables full infrastructure-as-code. Manual deployment (A) isn't repeatable. Shell scripts (C) lack state management. Separate teams (D) create coordination overhead.

**Key Concept:** [Config Connector](https://cloud.google.com/config-connector/docs/overview)
</details>

### Question 23
**Scenario:** An application deployed on Cloud Run needs to access a Cloud SQL database. The connection should be secure without exposing the database to the public internet. How should this be configured?

A. Whitelist Cloud Run's IP range in Cloud SQL firewall
B. Use Cloud SQL Auth Proxy as a sidecar with private IP and VPC connector
C. Enable public IP on Cloud SQL and use SSL
D. Store database credentials in environment variables

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Run can use VPC connectors to access resources on private IPs. Cloud SQL Auth Proxy handles authentication and encrypts the connection. This keeps the database on private IP only (no public exposure) while enabling secure access. IP whitelisting (A) requires public IP. Public IP with SSL (C) exposes the database to internet. Credentials in env vars (D) addresses authentication, not network security.

**Key Concept:** [Cloud Run VPC Connector](https://cloud.google.com/run/docs/configuring/connecting-vpc)
</details>

### Question 24
**Scenario:** A company requires that all deployments to production are approved by a security team and have passed all tests. They want an audit trail of who approved each deployment. How should this be implemented?

A. Verbal approval before deployment
B. Cloud Deploy with approval requirements for production target and IAM logging
C. Email approval threads
D. Shared production credentials for approved deployers

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Deploy supports approval workflows where specific roles must approve before promotion to production. All approvals are logged with IAM identity, providing audit trail. This integrates with existing IAM for authorization. Verbal (A) and email (C) approvals aren't enforceable or auditable. Shared credentials (D) eliminate accountability.

**Key Concept:** [Cloud Deploy Approvals](https://cloud.google.com/deploy/docs/promote-release)
</details>

### Question 25
**Scenario:** An organization uses GitOps for deployments - all changes go through Git pull requests. They need the actual state of their GKE clusters to match the desired state in Git automatically. What tool provides this?

A. kubectl apply from CI/CD
B. Config Sync with a Git repository as source of truth
C. Helm charts stored in Git
D. Cloud Deploy with manual promotion

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Config Sync continuously synchronizes Kubernetes configurations from a Git repository to GKE clusters. It detects drift and automatically reconciles, ensuring actual state matches desired state in Git. This is the GitOps model. kubectl in CI/CD (A) only applies on pipeline runs, not continuously. Helm (C) is a packaging format, not a sync mechanism. Cloud Deploy (D) requires manual or triggered promotion.

**Key Concept:** [Config Sync](https://cloud.google.com/anthos-config-management/docs/config-sync-overview)
</details>

---

## Domain 4: Integrating Google Cloud Services (Questions 26-34)

### Question 26
**Scenario:** An application needs to generate PDF reports from templates with customer data. Report generation can take 30-60 seconds. Users should be able to request a report and receive it via email without waiting. What architecture supports this?

A. Synchronous Cloud Function that generates and returns the PDF
B. Cloud Tasks triggering a Cloud Run job for PDF generation, storing result in Cloud Storage, sending email via SendGrid
C. Cron job generating all reports nightly
D. Client-side PDF generation

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Tasks enables asynchronous processing - the request is accepted immediately while processing happens in the background. Cloud Run jobs can run longer tasks (up to 60 minutes). The completed PDF is stored in Cloud Storage with a signed URL sent via email. This provides good user experience without timeout issues. Synchronous function (A) risks timeouts. Nightly cron (C) doesn't support on-demand. Client-side (D) exposes data and logic.

**Key Concept:** [Cloud Tasks](https://cloud.google.com/tasks/docs/dual-overview)
</details>

### Question 27
**Scenario:** A company needs to build a chatbot that can answer questions about their product catalog stored in Firestore. The bot should understand natural language and provide relevant responses. What GCP services support this?

A. Build custom NLP models from scratch
B. Dialogflow CX with Firestore integration for fulfillment
C. Cloud Functions responding to keyword matching
D. BigQuery for natural language queries

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Dialogflow CX provides natural language understanding, intent detection, and conversation management. Webhook fulfillment can query Firestore to retrieve product information based on detected intents. This combines conversational AI with data retrieval. Custom NLP (A) requires ML expertise and data. Keyword matching (C) doesn't understand natural language. BigQuery (D) is for analytics, not conversational interfaces.

**Key Concept:** [Dialogflow CX](https://cloud.google.com/dialogflow/cx/docs/basics)
</details>

### Question 28
**Scenario:** An application needs to process uploaded images by: detecting objects in the image, extracting any text, and storing the results. This should happen automatically when images are uploaded to Cloud Storage. What architecture implements this?

A. Scheduled batch processing of all images
B. Cloud Storage trigger → Cloud Function → Vision AI (object detection, OCR) → results to Firestore
C. Manual processing through Cloud Console
D. Client-side image processing before upload

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Storage Eventarc triggers invoke Cloud Functions on object creation. Vision AI provides object detection and OCR capabilities via simple API calls. Results stored in Firestore enable querying and retrieval. This is event-driven and fully automated. Batch processing (A) adds delay. Manual processing (C) doesn't scale. Client-side (D) is unreliable and slow on mobile devices.

**Key Concept:** [Vision AI](https://cloud.google.com/vision/docs/features-list)
</details>

### Question 29
**Scenario:** A retail application needs to provide real-time inventory updates to web and mobile clients. When inventory changes, all connected clients should see the update within 1 second. What approach provides this real-time capability?

A. Clients poll the API every second
B. Firestore real-time listeners with client SDKs
C. WebSocket server on Compute Engine
D. Server-sent events from Cloud Functions

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Firestore provides built-in real-time synchronization. Client SDKs (web, iOS, Android) can subscribe to document or collection changes, receiving updates pushed from Firestore automatically when data changes. No custom infrastructure needed. Polling (A) is inefficient and delays updates. Custom WebSocket (C) requires infrastructure management. Cloud Functions (D) don't maintain persistent connections.

**Key Concept:** [Firestore Real-time Updates](https://cloud.google.com/firestore/docs/query-data/listen)
</details>

### Question 30
**Scenario:** A data processing pipeline needs to read from Pub/Sub, transform records, aggregate by key over time windows, and write to BigQuery. Transformations include calling an external API for enrichment. What service is BEST for this?

A. Cloud Functions triggered by Pub/Sub
B. Dataflow with Apache Beam SDK
C. Cloud Run processing Pub/Sub messages
D. BigQuery scheduled queries

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Dataflow with Apache Beam provides streaming data processing with built-in windowing functions, exactly-once processing, and managed autoscaling. Beam supports calling external APIs within transforms. Dataflow handles the complexity of distributed streaming. Cloud Functions (A) don't support windowing. Cloud Run (C) can process messages but lacks built-in streaming primitives. BigQuery (D) can't call external APIs.

**Key Concept:** [Dataflow Streaming](https://cloud.google.com/dataflow/docs/concepts/streaming-pipelines)
</details>

### Question 31
**Scenario:** An application needs to schedule a task to run at a specific time in the future (e.g., send a reminder email 24 hours before an appointment). The scheduled time is determined at runtime for each task. What service handles this?

A. Cloud Scheduler with dynamic cron expressions
B. Cloud Tasks with scheduled delivery time
C. Cloud Functions with setTimeout
D. Store tasks in database and poll

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Tasks allows specifying a scheduleTime for each task, delivering it to the handler at that specific time. This supports dynamic, per-task scheduling determined at runtime. Cloud Scheduler (A) is for recurring schedules, not one-time future execution. setTimeout in Functions (C) doesn't survive function termination. Database polling (D) is inefficient and requires always-on compute.

**Key Concept:** [Cloud Tasks Scheduling](https://cloud.google.com/tasks/docs/creating-http-target-tasks)
</details>

### Question 32
**Scenario:** A healthcare application needs to redact PII (names, phone numbers, medical record numbers) from clinical notes before storing them for research. The redaction should be automated and comprehensive. What GCP service provides this?

A. Regular expressions in application code
B. Cloud Data Loss Prevention (DLP) API with de-identification transforms
C. Manual review of all documents
D. Encrypt documents instead of redaction

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud DLP detects sensitive information types including PII and supports de-identification transforms (redaction, masking, tokenization). It handles healthcare-specific identifiers and can process documents at scale. Custom regex (A) misses variations and new patterns. Manual review (C) doesn't scale. Encryption (D) preserves data but doesn't enable research use of redacted content.

**Key Concept:** [Cloud DLP De-identification](https://cloud.google.com/dlp/docs/deidentify-sensitive-data)
</details>

### Question 33
**Scenario:** An e-commerce application needs to send transactional emails (order confirmations, shipping updates) reliably. The system should handle high volumes during sales events and provide delivery tracking. How should this be implemented?

A. Direct SMTP from application servers
B. Cloud Tasks queuing email requests to a Cloud Run service using SendGrid API with delivery webhooks to Pub/Sub
C. Store emails in database for batch sending
D. Client-side email sending

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Tasks provides reliable queueing with retry logic. SendGrid (or similar services) handles email delivery at scale with delivery/open/click tracking. Webhooks push delivery status to Pub/Sub for processing and analytics. This is reliable and scalable. Direct SMTP (A) from cloud IPs often gets blocked. Batch sending (C) delays transactional emails. Client-side (D) exposes email infrastructure.

**Key Concept:** [Cloud Tasks with External Services](https://cloud.google.com/tasks/docs/creating-http-target-tasks)
</details>

### Question 34
**Scenario:** An application needs to generate custom product recommendations for users based on their browsing history and purchase patterns. The recommendation model should be trained on historical data and serve predictions with low latency. What approach is recommended?

A. Rule-based recommendations with hard-coded logic
B. Vertex AI Recommendations AI or custom model on Vertex AI with online prediction endpoints
C. Pre-compute all recommendations nightly
D. Third-party recommendation SaaS only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Vertex AI Recommendations AI provides a managed service for product recommendations, handling model training and serving. Alternatively, custom models can be trained and deployed to Vertex AI endpoints for low-latency predictions. This leverages ML for personalization. Rule-based (A) doesn't capture complex patterns. Nightly pre-compute (C) misses real-time behavior. Third-party only (D) may not integrate as well with GCP data.

**Key Concept:** [Recommendations AI](https://cloud.google.com/recommendations-ai/docs/overview)
</details>

---

## Domain 5: Managing Deployed Applications (Questions 35-40)

### Question 35
**Scenario:** A production application is experiencing intermittent latency spikes. The team needs to identify which service and code path is causing the slowdowns. Standard metrics show overall latency but not the root cause. What tool helps diagnose this?

A. Cloud Monitoring dashboards only
B. Cloud Trace to analyze individual request latency with Cloud Profiler for code-level analysis
C. Application logs searching for slow operations
D. User feedback surveys

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Trace captures distributed traces showing latency breakdown across services for individual requests. Filtering for slow traces reveals which service causes delays. Cloud Profiler shows CPU and memory usage by code path, identifying expensive functions. Together they pinpoint the root cause. Dashboards (A) show aggregates. Logs (C) may not have timing info. User feedback (D) is too late.

**Key Concept:** [Cloud Profiler](https://cloud.google.com/profiler/docs/about-profiler)
</details>

### Question 36
**Scenario:** A team needs to ensure their API maintains backward compatibility as it evolves. They want to detect breaking changes before deployment. What practice helps ensure API stability?

A. No API documentation
B. API versioning with automated contract testing using tools like Protolock for gRPC or OpenAPI diff
C. Allow any changes with client notification
D. Never change the API

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** API versioning (v1, v2) allows evolution while maintaining old versions. Automated contract testing detects breaking changes - Protolock for gRPC protobuf schemas, OpenAPI diff for REST APIs. These can be integrated into CI to fail builds with breaking changes. No documentation (A) makes compatibility impossible to track. Any changes (C) breaks clients. Never changing (D) prevents improvement.

**Key Concept:** [API Versioning Best Practices](https://cloud.google.com/apis/design/versioning)
</details>

### Question 37
**Scenario:** An application running on Cloud Run needs to gracefully handle instance termination during scale-down. In-flight requests should complete, and cleanup tasks should run before shutdown. How is this achieved?

A. Ignore termination signals
B. Handle SIGTERM signal with graceful shutdown logic and configure appropriate request timeout
C. Use synchronous processing only
D. Disable autoscaling

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Run sends SIGTERM before termination, allowing up to 10 seconds (configurable) for graceful shutdown. Applications should handle SIGTERM, stop accepting new requests, complete in-flight requests, and run cleanup. Setting appropriate request timeout ensures requests complete. Ignoring signals (A) causes abrupt termination. Disabling autoscaling (D) prevents cost optimization.

**Key Concept:** [Cloud Run Container Lifecycle](https://cloud.google.com/run/docs/container-contract#container-lifecycle)
</details>

### Question 38
**Scenario:** A team manages multiple microservices across development, staging, and production environments. They need to track which version of each service is deployed in each environment and quickly identify configuration differences. How should this be managed?

A. Spreadsheet tracking deployments
B. Cloud Deploy release inventory with Cloud Asset Inventory for resource configuration
C. Memory and tribal knowledge
D. Check each service individually

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Deploy tracks releases across targets (environments), showing which version is deployed where. Cloud Asset Inventory captures resource configurations across projects, enabling diff analysis between environments. This provides automated, accurate tracking. Spreadsheets (A) become outdated. Tribal knowledge (C) isn't reliable. Checking individually (D) is time-consuming and error-prone.

**Key Concept:** [Cloud Asset Inventory](https://cloud.google.com/asset-inventory/docs/overview)
</details>

### Question 39
**Scenario:** An application's error rate suddenly increased after a deployment. The team needs to quickly identify which errors are new (introduced by the deployment) versus existing errors. They use Cloud Error Reporting. How can they make this determination?

A. Read all error logs manually
B. Use Error Reporting's "first seen" and "last seen" timestamps, filter by service version
C. Assume all errors are from the new deployment
D. Wait for user reports

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Error Reporting groups similar errors and tracks first/last occurrence. Filtering by service version (set via labels) shows errors specific to the new version. "First seen" timestamps after deployment indicate new errors. Errors seen before deployment are pre-existing. This enables rapid triage. Manual log reading (A) is slow. Assuming (C) may cause false positives. User reports (D) are delayed.

**Key Concept:** [Error Reporting](https://cloud.google.com/error-reporting/docs/viewing-errors)
</details>

### Question 40
**Scenario:** A production system needs to maintain 99.9% availability SLO. The team needs to track error budget consumption, get alerts when approaching budget exhaustion, and make data-driven decisions about deployments vs. reliability work. What should they implement?

A. Manual availability tracking in spreadsheets
B. Cloud Monitoring SLOs with error budget alerts and burn rate policies
C. Monitor uptime only
D. Ignore SLOs and deploy whenever

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Monitoring SLOs define service level objectives based on metrics (availability, latency). Error budgets are automatically calculated. Burn rate alerts notify when error budget is being consumed too quickly (e.g., 10x normal rate). This enables informed trade-offs between feature velocity and reliability. Spreadsheets (A) can't alert. Uptime only (C) misses other SLI types. Ignoring SLOs (D) risks reliability.

**Key Concept:** [Cloud Monitoring SLOs](https://cloud.google.com/monitoring/sli-slo-concepts)
</details>
