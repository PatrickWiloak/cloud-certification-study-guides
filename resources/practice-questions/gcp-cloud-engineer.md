# GCP Associate Cloud Engineer Practice Questions

40 practice questions covering all exam domains.

**Exam Domain Breakdown:**
- Setting up a cloud solution environment (17.5%) - 7 questions
- Planning and configuring a cloud solution (17.5%) - 7 questions
- Deploying and implementing a cloud solution (25%) - 10 questions
- Ensuring successful operation of a cloud solution (20%) - 8 questions
- Configuring access and security (20%) - 8 questions

---

## Setting Up a Cloud Solution Environment (17.5%)

### Question 1
**Scenario:** A company is starting to use Google Cloud and needs to organize resources for three departments: Marketing, Engineering, and Sales. Each department should have separate billing and resource management. What is the recommended approach?

A. Create one project with three folders
B. Create three separate projects
C. Create one organization with three folders, each containing projects
D. Create three billing accounts with shared projects

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** The recommended approach is to use an organization with folders for each department. Folders provide grouping for policy inheritance and management, while projects within folders enable separate resource management. This follows GCP's resource hierarchy best practices. Single projects don't provide isolation. Folders can't exist at the project level. Shared projects with multiple billing accounts create complexity.

**Key Concept:** [Resource Hierarchy](https://cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy)
</details>

---

### Question 2
**Scenario:** A developer needs to use the gcloud CLI to manage resources in a specific project. They have access to multiple projects. How should they configure gcloud to work with the correct project?

A. Set the GOOGLE_PROJECT environment variable
B. Run `gcloud config set project PROJECT_ID`
C. Edit the ~/.gcloud/config file manually
D. Add --project flag to every command

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** `gcloud config set project PROJECT_ID` sets the default project for all subsequent gcloud commands. While --project flag works, it's not efficient for repeated use. The environment variable is GOOGLE_CLOUD_PROJECT, not GOOGLE_PROJECT. Manual config editing is not recommended.

**Key Concept:** [gcloud CLI Configuration](https://cloud.google.com/sdk/gcloud/reference/config/set)
</details>

---

### Question 3
**Scenario:** A company needs to grant a contractor temporary access to view Cloud Storage buckets in a specific project for an audit. The access should expire in 30 days. What should be configured?

A. Add the contractor to a Google Group with storage.objectViewer role
B. Grant storage.objectViewer role with an IAM condition for time-based access
C. Create a service account and share credentials
D. Grant storage.admin role temporarily

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** IAM conditions allow time-based access expiration. Granting the storage.objectViewer role with a condition specifying access expiration after 30 days meets the requirements. Google Groups don't expire. Service account credentials can be shared indefinitely. Admin role grants more than necessary.

**Key Concept:** [IAM Conditions](https://cloud.google.com/iam/docs/conditions-overview)
</details>

---

### Question 4
**Scenario:** An organization needs to ensure that no resources are created in certain regions due to data residency requirements. How can they enforce this across all projects?

A. Configure VPC firewall rules
B. Set organization policies with location constraints
C. Use Cloud Armor policies
D. Configure Cloud NAT for specific regions

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Organization policies with location constraints (resource location restriction) enforce where resources can be created across the organization. Firewall rules control network traffic, not resource creation. Cloud Armor protects applications. Cloud NAT provides outbound connectivity.

**Key Concept:** [Organization Policies](https://cloud.google.com/resource-manager/docs/organization-policy/overview)
</details>

---

### Question 5
**Scenario:** A team needs to create multiple identical environments (dev, staging, prod) with the same infrastructure configuration. What tool should they use?

A. Cloud Console
B. Deployment Manager or Terraform
C. gcloud CLI scripts only
D. Cloud Marketplace

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Deployment Manager (GCP-native) or Terraform (multi-cloud) provides infrastructure as code for creating repeatable, version-controlled environments. Console is manual. CLI scripts lack state management and idempotency. Marketplace is for pre-built solutions.

**Key Concept:** [Cloud Deployment Manager](https://cloud.google.com/deployment-manager/docs)
</details>

---

### Question 6
**Scenario:** A company wants to monitor and control cloud spending with alerts when costs exceed budgets. What should they configure?

A. Cloud Monitoring alerts
B. Cloud Billing budgets and alerts
C. Quotas and limits
D. Organization policies

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Billing budgets allow setting spending thresholds with alerts when actual or forecasted spending approaches or exceeds the budget. Cloud Monitoring tracks performance, not costs. Quotas limit resource creation. Organization policies enforce compliance rules.

**Key Concept:** [Cloud Billing Budgets](https://cloud.google.com/billing/docs/how-to/budgets)
</details>

---

### Question 7
**Scenario:** An administrator needs to view all IAM policy changes across the organization for compliance auditing. Where should they look?

A. Cloud Monitoring
B. Cloud Audit Logs (Admin Activity logs)
C. Cloud Billing reports
D. Security Command Center

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Audit Logs, specifically Admin Activity logs, record all IAM policy changes and other administrative actions. These logs are always on and can't be disabled. Cloud Monitoring tracks performance. Billing tracks costs. Security Command Center provides security findings.

**Key Concept:** [Cloud Audit Logs](https://cloud.google.com/logging/docs/audit)
</details>

---

## Planning and Configuring a Cloud Solution (17.5%)

### Question 8
**Scenario:** A web application needs to automatically scale based on CPU usage. The application runs on Compute Engine VMs. What should be configured?

A. Cloud Run with automatic scaling
B. Managed Instance Group with autoscaling policy
C. Kubernetes HPA
D. Cloud Functions with concurrent execution

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Managed Instance Groups (MIGs) with autoscaling policies automatically add or remove VMs based on metrics like CPU usage. Cloud Run is for containers. Kubernetes requires GKE cluster. Cloud Functions is for event-driven code.

**Key Concept:** [Compute Engine Autoscaling](https://cloud.google.com/compute/docs/autoscaler)
</details>

---

### Question 9
**Scenario:** A company needs to store 100 TB of data that will be accessed about once per year for compliance purposes. Cost is the primary concern. What storage option should they choose?

A. Cloud Storage Standard
B. Cloud Storage Nearline
C. Cloud Storage Coldline
D. Cloud Storage Archive

<details>
<summary>Answer</summary>

**Correct: D**

**Why:** Cloud Storage Archive is the lowest-cost option for data accessed less than once per year. Nearline is for monthly access. Coldline is for quarterly access. Standard is for frequently accessed data. Archive's minimum storage duration (365 days) matches the annual access pattern.

**Key Concept:** [Cloud Storage Classes](https://cloud.google.com/storage/docs/storage-classes)
</details>

---

### Question 10
**Scenario:** An application needs a fully managed relational database with automatic replication across multiple regions for high availability and disaster recovery. What should be used?

A. Cloud SQL with read replicas
B. Cloud Spanner
C. Firestore
D. BigQuery

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Spanner provides globally distributed, strongly consistent relational database with automatic replication across regions. Cloud SQL read replicas don't provide multi-region write capability. Firestore is NoSQL. BigQuery is for analytics.

**Key Concept:** [Cloud Spanner](https://cloud.google.com/spanner/docs)
</details>

---

### Question 11
**Scenario:** A data analytics team needs to run SQL queries on petabytes of data without managing infrastructure. The queries are ad-hoc and run a few times per day. What service should they use?

A. Cloud SQL
B. Cloud Spanner
C. BigQuery
D. Dataproc

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** BigQuery is a serverless data warehouse designed for analytics on large datasets with on-demand pricing for ad-hoc queries. Cloud SQL and Spanner are for transactional workloads. Dataproc requires cluster management for Spark/Hadoop.

**Key Concept:** [BigQuery](https://cloud.google.com/bigquery/docs)
</details>

---

### Question 12
**Scenario:** A company needs to connect their on-premises network to Google Cloud with dedicated, private connectivity for regulatory compliance. What should they use?

A. Cloud VPN
B. Cloud Interconnect
C. Cloud Router
D. VPC Peering

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Interconnect provides dedicated, private connectivity between on-premises and Google Cloud, not traversing the public internet. Cloud VPN uses encrypted tunnels over the internet. Cloud Router enables dynamic routing. VPC Peering connects VPCs within GCP.

**Key Concept:** [Cloud Interconnect](https://cloud.google.com/network-connectivity/docs/interconnect)
</details>

---

### Question 13
**Scenario:** An application needs to send 10 million push notifications per day with minimal latency. The notifications must be processed exactly once. What GCP service should be used?

A. Cloud Tasks
B. Pub/Sub
C. Cloud Functions
D. Cloud Run

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Pub/Sub can handle millions of messages with exactly-once delivery guarantees and integrates with push notification services. Cloud Tasks is for task queuing. Cloud Functions and Cloud Run are compute services, not messaging.

**Key Concept:** [Cloud Pub/Sub](https://cloud.google.com/pubsub/docs)
</details>

---

### Question 14
**Scenario:** A startup needs to deploy a containerized web application quickly without managing servers or Kubernetes clusters. They need automatic scaling to zero when idle to minimize costs. What should they use?

A. Compute Engine
B. GKE Standard
C. Cloud Run
D. App Engine Flexible

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Cloud Run is a serverless container platform that automatically scales, including to zero, with pay-per-use pricing. Compute Engine requires VM management. GKE requires cluster management. App Engine Flexible doesn't scale to zero.

**Key Concept:** [Cloud Run](https://cloud.google.com/run/docs)
</details>

---

## Deploying and Implementing a Cloud Solution (25%)

### Question 15
**Scenario:** A developer needs to deploy a Compute Engine VM from a custom image. The VM should automatically restart if it fails. How should this be configured?

A. Create an instance template and unmanaged instance group
B. Create an instance template and managed instance group
C. Create a single VM with automatic restart enabled
D. Use Kubernetes with node auto-repair

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** A managed instance group (MIG) with an instance template automatically recreates failed VMs to maintain the desired count. While single VMs can restart, MIGs provide better reliability. Unmanaged instance groups don't auto-heal. Kubernetes is for containers.

**Key Concept:** [Managed Instance Groups](https://cloud.google.com/compute/docs/instance-groups/creating-groups-of-managed-instances)
</details>

---

### Question 16
**Scenario:** A developer needs to deploy a containerized application to GKE. The application needs 2 replicas with a load balancer. What Kubernetes resources should be created?

A. Deployment and Service (type: LoadBalancer)
B. Pod and Service (type: ClusterIP)
C. StatefulSet and Ingress
D. DaemonSet and NodePort Service

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** A Deployment manages replicas (2 pods), and a Service with type LoadBalancer creates a GCP load balancer to distribute traffic. Pods alone don't provide replica management. ClusterIP is internal only. StatefulSet is for stateful apps. DaemonSet runs on all nodes.

**Key Concept:** [GKE Deployments and Services](https://cloud.google.com/kubernetes-engine/docs/concepts/service)
</details>

---

### Question 17
**Scenario:** A developer needs to store application configuration that can be updated without redeploying the application. The configuration contains non-sensitive data like feature flags. What should they use?

A. Environment variables
B. Cloud Storage bucket
C. Secret Manager
D. Runtime Configurator (Deployment Manager)

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Storing configuration in Cloud Storage allows updates without redeployment. Applications can periodically check for changes. Environment variables require redeployment. Secret Manager is for sensitive data. Runtime Configurator is deprecated.

**Key Concept:** [Application Configuration](https://cloud.google.com/solutions/configuring-apps-on-gcp)
</details>

---

### Question 18
**Scenario:** A company needs to copy 50 TB of data from AWS S3 to Cloud Storage. They want to minimize network costs and time. What should they use?

A. gsutil cp command
B. Storage Transfer Service
C. Transfer Appliance
D. BigQuery Data Transfer Service

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Storage Transfer Service is optimized for large-scale transfers from other cloud providers like AWS S3, with scheduling and bandwidth management. gsutil is for smaller transfers. Transfer Appliance is for on-premises data. BigQuery Data Transfer is for analytics data sources.

**Key Concept:** [Storage Transfer Service](https://cloud.google.com/storage-transfer-service)
</details>

---

### Question 19
**Scenario:** A developer needs to create a Cloud SQL PostgreSQL instance with high availability within a region. What configuration is required?

A. Create two instances with manual failover
B. Enable regional availability (HA) configuration
C. Create read replicas in different zones
D. Use Cloud SQL Proxy with multiple endpoints

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud SQL high availability (HA) configuration creates a primary and standby instance in different zones with automatic failover. Manual instances don't auto-failover. Read replicas are for read scaling, not HA. Cloud SQL Proxy is for secure connections.

**Key Concept:** [Cloud SQL High Availability](https://cloud.google.com/sql/docs/postgres/high-availability)
</details>

---

### Question 20
**Scenario:** A developer needs to deploy a Cloud Function that triggers when a file is uploaded to a Cloud Storage bucket. What trigger type should be configured?

A. HTTP trigger
B. Cloud Storage trigger
C. Pub/Sub trigger
D. Cloud Scheduler trigger

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Functions supports Cloud Storage triggers that fire on object creation (finalize), deletion, and metadata updates. HTTP triggers respond to HTTP requests. Pub/Sub triggers respond to messages. Cloud Scheduler is for cron jobs.

**Key Concept:** [Cloud Functions Triggers](https://cloud.google.com/functions/docs/calling)
</details>

---

### Question 21
**Scenario:** A web application on GKE needs to serve HTTPS traffic with automatic SSL certificate management. What should be configured?

A. Network Load Balancer with manual certificates
B. GKE Ingress with managed certificates
C. Cloud CDN with custom certificates
D. Cloud Armor with SSL policies

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** GKE Ingress with Google-managed certificates automatically provisions and renews SSL certificates. Network Load Balancer operates at Layer 4 (no SSL termination). Cloud CDN caches content. Cloud Armor provides security rules.

**Key Concept:** [GKE Managed Certificates](https://cloud.google.com/kubernetes-engine/docs/how-to/managed-certs)
</details>

---

### Question 22
**Scenario:** A developer needs to deploy updates to a Cloud Run service with the ability to gradually shift traffic to the new revision and rollback if issues occur. What should they use?

A. Blue/green deployment with two services
B. Traffic splitting between revisions
C. Canary deployment with GKE
D. Instance group rolling update

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Run supports traffic splitting between revisions, allowing gradual rollout and instant rollback by adjusting traffic percentages. No need for separate services. GKE is not Cloud Run. Instance groups are for VMs.

**Key Concept:** [Cloud Run Traffic Management](https://cloud.google.com/run/docs/rollouts-rollbacks-traffic-migration)
</details>

---

### Question 23
**Scenario:** A company needs to run batch processing jobs on large datasets using Apache Spark without managing infrastructure. What should they use?

A. Compute Engine cluster
B. Dataproc
C. Dataflow
D. Cloud Composer

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Dataproc provides managed Spark and Hadoop clusters for batch processing with easy setup and teardown. Compute Engine requires manual cluster management. Dataflow is for streaming and batch with Apache Beam. Cloud Composer is for workflow orchestration.

**Key Concept:** [Dataproc](https://cloud.google.com/dataproc/docs)
</details>

---

### Question 24
**Scenario:** A developer needs to expose a set of microservices through a single API endpoint with authentication, rate limiting, and monitoring. What service should be used?

A. Cloud Load Balancing
B. Apigee API Platform
C. Cloud Endpoints
D. Traffic Director

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Cloud Endpoints provides API management including authentication, rate limiting, monitoring, and a single entry point for microservices. Apigee is enterprise-grade but more complex. Load Balancing distributes traffic. Traffic Director is a service mesh control plane.

**Key Concept:** [Cloud Endpoints](https://cloud.google.com/endpoints/docs)
</details>

---

## Ensuring Successful Operation of a Cloud Solution (20%)

### Question 25
**Scenario:** A Cloud SQL instance is experiencing slow query performance. What tool should be used to identify the problematic queries?

A. Cloud Monitoring dashboards
B. Query Insights
C. Cloud Trace
D. Cloud Profiler

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Query Insights provides query-level performance data including slow queries, query plans, and wait events. Cloud Monitoring shows instance-level metrics. Cloud Trace is for distributed tracing. Cloud Profiler analyzes application code performance.

**Key Concept:** [Cloud SQL Query Insights](https://cloud.google.com/sql/docs/postgres/query-insights-overview)
</details>

---

### Question 26
**Scenario:** An application running on GKE needs to be notified when CPU usage exceeds 80% so the team can investigate. What should be configured?

A. GKE cluster autoscaler
B. Cloud Monitoring alerting policy
C. Horizontal Pod Autoscaler
D. Kubernetes liveness probe

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Monitoring alerting policies can trigger notifications when metrics exceed thresholds. Autoscaler and HPA automatically adjust resources but don't notify humans. Liveness probes check container health for Kubernetes.

**Key Concept:** [Cloud Monitoring Alerting](https://cloud.google.com/monitoring/alerts)
</details>

---

### Question 27
**Scenario:** A Cloud Function is failing intermittently. The developer needs to view the error messages and stack traces. Where should they look?

A. Cloud Monitoring metrics
B. Cloud Logging
C. Cloud Trace
D. Cloud Debugger

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Logging captures function logs including errors and stack traces. Cloud Monitoring shows metrics. Cloud Trace shows request latency. Cloud Debugger inspects running code state.

**Key Concept:** [Cloud Functions Logging](https://cloud.google.com/functions/docs/monitoring/logging)
</details>

---

### Question 28
**Scenario:** A company needs to create automated backups of a Cloud SQL database and retain them for 30 days. What should be configured?

A. Cloud Storage lifecycle policies
B. Cloud SQL automated backups with retention settings
C. Scheduled Cloud Functions to export data
D. Compute Engine snapshots

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud SQL provides automated backups with configurable retention (up to 365 days). Storage lifecycle is for objects. Cloud Functions export would be manual automation. Snapshots are for disk volumes.

**Key Concept:** [Cloud SQL Backups](https://cloud.google.com/sql/docs/postgres/backup-recovery/backups)
</details>

---

### Question 29
**Scenario:** A managed instance group needs to be updated to a new VM image with minimal disruption. The update should replace instances gradually. What update type should be used?

A. Opportunistic update
B. Proactive rolling update
C. Blue/green deployment
D. Recreate update

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Proactive rolling updates gradually replace instances with new ones, ensuring availability during the update. Opportunistic waits for instances to be recreated for other reasons. Blue/green creates parallel environments. Recreate stops all before starting new ones.

**Key Concept:** [MIG Update Policies](https://cloud.google.com/compute/docs/instance-groups/rolling-out-updates-to-managed-instance-groups)
</details>

---

### Question 30
**Scenario:** A GKE workload needs to read data from Cloud Storage. What is the recommended way to grant access without storing credentials in the container?

A. Store service account key in a Kubernetes secret
B. Use Workload Identity
C. Grant permissions to the default compute service account
D. Use Cloud SQL Auth proxy

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Workload Identity allows Kubernetes service accounts to act as Google service accounts without managing keys. Storing keys in secrets is a security risk. Default compute SA grants broad access. Cloud SQL Auth proxy is for database connections.

**Key Concept:** [Workload Identity](https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity)
</details>

---

### Question 31
**Scenario:** A company needs to track costs across different teams using the same project. What should they implement?

A. Separate billing accounts
B. Labels on resources
C. Multiple service accounts
D. Resource quotas

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Labels allow tagging resources with team identifiers, enabling cost breakdown in billing reports. Separate billing accounts require separate projects. Service accounts are for authentication. Quotas limit resource usage, not track costs.

**Key Concept:** [Resource Labels](https://cloud.google.com/compute/docs/labeling-resources)
</details>

---

### Question 32
**Scenario:** A Compute Engine VM is running slowly. The administrator needs to analyze CPU, memory, and disk performance. What tool should they use?

A. Cloud Monitoring metrics explorer
B. Serial console output
C. VM instance details page
D. Cloud Shell

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Cloud Monitoring metrics explorer provides detailed CPU, memory, disk, and network metrics with visualization. Serial console shows boot logs. Instance details show configuration. Cloud Shell is a terminal.

**Key Concept:** [Compute Engine Monitoring](https://cloud.google.com/compute/docs/monitoring)
</details>

---

## Configuring Access and Security (20%)

### Question 33
**Scenario:** A developer needs to grant a service account permission to read objects from a specific Cloud Storage bucket but not other buckets. What is the recommended approach?

A. Grant storage.objectViewer role at project level
B. Grant storage.objectViewer role at bucket level
C. Grant storage.admin role at project level
D. Add the service account to allUsers

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Granting roles at the bucket level (resource level) follows least privilege, giving access only to the specific bucket. Project-level grants access to all buckets. Admin role grants more than needed. allUsers is for public access.

**Key Concept:** [Cloud Storage IAM](https://cloud.google.com/storage/docs/access-control/iam)
</details>

---

### Question 34
**Scenario:** A company needs to ensure all service account keys are rotated every 90 days. How can they enforce this?

A. Organization policy
B. IAM Conditions
C. Cloud Asset Inventory with monitoring
D. VPC Service Controls

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Cloud Asset Inventory can track service account key age, and monitoring alerts can notify when keys exceed 90 days. There's no organization policy for key rotation. IAM conditions control access timing. VPC Service Controls protect data.

**Key Concept:** [Service Account Key Management](https://cloud.google.com/iam/docs/best-practices-for-managing-service-account-keys)
</details>

---

### Question 35
**Scenario:** A Cloud Run service needs to access a Cloud SQL database. The service should authenticate securely without managing passwords. What should be configured?

A. Store password in Secret Manager
B. Use Cloud SQL Auth Proxy with IAM database authentication
C. Whitelist Cloud Run IP addresses
D. Use VPC connector with public IP

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud SQL Auth Proxy with IAM database authentication allows services to connect using IAM identity instead of passwords. Secret Manager stores credentials but requires rotation. IP whitelisting doesn't authenticate. VPC connector provides connectivity, not authentication.

**Key Concept:** [Cloud SQL IAM Authentication](https://cloud.google.com/sql/docs/postgres/iam-authentication)
</details>

---

### Question 36
**Scenario:** A company wants to prevent data exfiltration from their GCP project to external destinations. What should they implement?

A. Cloud Armor policies
B. VPC Service Controls
C. Cloud NAT
D. Firewall rules

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** VPC Service Controls create security perimeters around GCP resources, preventing data from leaving defined boundaries. Cloud Armor protects applications. Cloud NAT provides outbound connectivity. Firewall rules control network traffic but not API-level data exfiltration.

**Key Concept:** [VPC Service Controls](https://cloud.google.com/vpc-service-controls/docs)
</details>

---

### Question 37
**Scenario:** An application needs to encrypt data using a customer-managed encryption key (CMEK). Where should the key be created and managed?

A. Cloud Storage
B. Cloud KMS
C. Secret Manager
D. IAM

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud KMS (Key Management Service) is purpose-built for creating and managing encryption keys used for CMEK across GCP services. Cloud Storage uses keys, not manages them. Secret Manager stores secrets. IAM manages access.

**Key Concept:** [Cloud KMS](https://cloud.google.com/kms/docs)
</details>

---

### Question 38
**Scenario:** A developer accidentally committed a service account key to a public GitHub repository. What is the first action they should take?

A. Delete the GitHub repository
B. Disable or delete the service account key
C. Revoke all user access to the project
D. Create a new service account

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Immediately disable or delete the compromised key to prevent unauthorized access. Deleting the repo doesn't revoke the key. Revoking user access doesn't address the leaked key. Creating new SA doesn't disable the compromised key.

**Key Concept:** [Compromised Credentials](https://cloud.google.com/iam/docs/best-practices-for-managing-service-account-keys#compromised)
</details>

---

### Question 39
**Scenario:** A GKE cluster needs to restrict which container images can be deployed. Only images from the company's Artifact Registry should be allowed. What should be configured?

A. Kubernetes RBAC
B. Binary Authorization
C. Pod Security Policy
D. Network Policy

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Binary Authorization enforces deploy-time security by requiring images to be signed or from approved registries. RBAC controls who can deploy. Pod Security Policy (deprecated) controls pod configurations. Network Policy controls network traffic.

**Key Concept:** [Binary Authorization](https://cloud.google.com/binary-authorization/docs)
</details>

---

### Question 40
**Scenario:** A company needs to audit who accessed specific BigQuery tables containing sensitive data. What should be enabled?

A. Cloud Audit Logs - Data Access logs
B. Cloud Monitoring metrics
C. BigQuery INFORMATION_SCHEMA
D. VPC Flow Logs

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Data Access audit logs record who accessed BigQuery data (reads and writes). Admin Activity logs don't capture data reads. INFORMATION_SCHEMA shows metadata. VPC Flow Logs capture network traffic.

**Key Concept:** [BigQuery Audit Logs](https://cloud.google.com/bigquery/docs/reference/auditlogs)
</details>

---

## Answer Key

| Question | Answer | Domain |
|----------|--------|--------|
| 1 | C | Setting Up Environment |
| 2 | B | Setting Up Environment |
| 3 | B | Setting Up Environment |
| 4 | B | Setting Up Environment |
| 5 | B | Setting Up Environment |
| 6 | B | Setting Up Environment |
| 7 | B | Setting Up Environment |
| 8 | B | Planning & Configuring |
| 9 | D | Planning & Configuring |
| 10 | B | Planning & Configuring |
| 11 | C | Planning & Configuring |
| 12 | B | Planning & Configuring |
| 13 | B | Planning & Configuring |
| 14 | C | Planning & Configuring |
| 15 | B | Deploying & Implementing |
| 16 | A | Deploying & Implementing |
| 17 | B | Deploying & Implementing |
| 18 | B | Deploying & Implementing |
| 19 | B | Deploying & Implementing |
| 20 | B | Deploying & Implementing |
| 21 | B | Deploying & Implementing |
| 22 | B | Deploying & Implementing |
| 23 | B | Deploying & Implementing |
| 24 | C | Deploying & Implementing |
| 25 | B | Ensuring Operation |
| 26 | B | Ensuring Operation |
| 27 | B | Ensuring Operation |
| 28 | B | Ensuring Operation |
| 29 | B | Ensuring Operation |
| 30 | B | Ensuring Operation |
| 31 | B | Ensuring Operation |
| 32 | A | Ensuring Operation |
| 33 | B | Access & Security |
| 34 | C | Access & Security |
| 35 | B | Access & Security |
| 36 | B | Access & Security |
| 37 | B | Access & Security |
| 38 | B | Access & Security |
| 39 | B | Access & Security |
| 40 | A | Access & Security |

---

## Study Resources

- [Google Cloud Associate Cloud Engineer](https://cloud.google.com/certification/cloud-engineer)
- [Google Cloud Documentation](https://cloud.google.com/docs)
- [Cloud Skills Boost](https://www.cloudskillsboost.google/)
- [GCP Free Tier](https://cloud.google.com/free)
