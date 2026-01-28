# GCP Professional Cloud DevOps Engineer Practice Questions

40 scenario-based questions weighted by exam domain.

## Exam Domain Breakdown
| Domain | Weight | Questions |
|--------|--------|-----------|
| Bootstrapping a Google Cloud organization | 10% | 4 |
| Building and implementing CI/CD pipelines | 28% | 11 |
| Applying site reliability engineering practices | 28% | 11 |
| Implementing service monitoring strategies | 22% | 9 |
| Optimizing service performance | 12% | 5 |

---

## Domain 1: Bootstrapping a Google Cloud Organization (Questions 1-4)

### Question 1
**Scenario:** A company is setting up their Google Cloud organization for the first time. They need to implement a folder structure that supports separation between production and non-production workloads, allows different teams to manage their own projects, and enforces consistent security policies across all projects. What structure should they implement?

A. Single folder with all projects
B. Hierarchical folder structure with org policies inherited: Organization → Environment folders (prod/non-prod) → Team folders → Projects
C. Separate Google Cloud organizations for each team
D. Projects without folders at organization level

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Hierarchical folders enable policy inheritance - org policies set at higher levels apply to all children. Environment folders (prod/non-prod) allow different policies (e.g., stricter controls in prod). Team folders delegate project creation while inheriting parent policies. This provides both governance and team autonomy. Single folder (A) doesn't enable differentiated policies. Separate orgs (C) complicate cross-team collaboration. No folders (D) lacks hierarchy for policy management.

**Key Concept:** [Resource Hierarchy](https://cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy)
</details>

### Question 2
**Scenario:** A DevOps team needs to ensure that all projects in the organization have certain security configurations: VMs cannot have external IPs, Cloud Storage buckets must not be public, and only approved regions can be used. How should these restrictions be enforced?

A. Document requirements and train teams
B. Organization policies applied at the organization or folder level
C. Manual review of all resource creation
D. Post-deployment scanning and remediation

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Organization policies provide centralized, inherited enforcement of restrictions. Policies like `compute.vmExternalIpAccess`, `storage.publicAccessPrevention`, and `gcp.resourceLocations` prevent non-compliant resources from being created. Applied at org or folder level, they affect all projects below. Documentation (A) isn't enforceable. Manual review (C) doesn't scale. Post-deployment (D) is reactive and allows temporary non-compliance.

**Key Concept:** [Organization Policy Constraints](https://cloud.google.com/resource-manager/docs/organization-policy/org-policy-constraints)
</details>

### Question 3
**Scenario:** A company wants to automate the provisioning of new projects for development teams. Each project should have standardized IAM bindings, enabled APIs, and configured logging. The process should be triggered by an approved request system. What approach provides this?

A. Manual project creation by admins
B. Terraform with a project factory module triggered by CI/CD upon approved merge requests
C. Give all developers project creator permissions
D. Pre-create projects for anticipated needs

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Terraform project factory modules create projects with consistent configuration (IAM, APIs, logging, network). GitOps workflow means project requests are pull requests - approved merges trigger CI/CD to create projects. This provides audit trail, consistency, and automation. Manual creation (A) is slow and inconsistent. Broad permissions (C) lack governance. Pre-creation (D) wastes resources and can't predict exact needs.

**Key Concept:** [Project Factory Module](https://github.com/terraform-google-modules/terraform-google-project-factory)
</details>

### Question 4
**Scenario:** An organization needs to ensure that all resource changes across all projects are logged for compliance and that logs cannot be modified or deleted by project owners. How should this be configured?

A. Enable data access logs in each project
B. Create an organization-level log sink to a separate logging project with locked log bucket and org policy preventing sink deletion
C. Export logs to Cloud Storage in each project
D. Trust project owners to maintain logs

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Organization-level aggregated sinks capture logs from all projects. Routing to a dedicated logging project with restricted access ensures project owners can't delete logs. Locked log buckets prevent modification for the retention period. Org policies can prevent deletion of the sink. Per-project logging (A, C) can be modified by project owners. Trust (D) doesn't meet compliance requirements.

**Key Concept:** [Aggregated Sinks](https://cloud.google.com/logging/docs/export/aggregated_sinks)
</details>

---

## Domain 2: Building and Implementing CI/CD Pipelines (Questions 5-15)

### Question 5
**Scenario:** A development team needs a CI/CD pipeline that builds container images, runs tests, scans for vulnerabilities, and deploys to GKE. Only images that pass vulnerability scanning should be deployed to production. What GCP services provide this pipeline?

A. Cloud Build → Artifact Registry → kubectl apply
B. Cloud Build with Container Analysis → Artifact Registry → Binary Authorization → Cloud Deploy to GKE
C. Jenkins on Compute Engine → Docker Hub → GKE
D. Manual builds and deployments

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Build handles CI (build, test). Container Analysis scans images in Artifact Registry for vulnerabilities. Binary Authorization enforces that only attested (scanned, approved) images can be deployed to GKE. Cloud Deploy manages the deployment pipeline with promotion and rollback. This is a secure software supply chain. Option A lacks security scanning and deployment management. Jenkins/Docker Hub (C) doesn't integrate with Binary Authorization. Manual (D) doesn't scale.

**Key Concept:** [Software Supply Chain Security](https://cloud.google.com/software-supply-chain-security/docs/overview)
</details>

### Question 6
**Scenario:** A team uses a monorepo containing multiple microservices. They want Cloud Build to only build and deploy services that have changed, not the entire repository on every commit. How can this be achieved?

A. Build everything on every commit
B. Use Cloud Build triggers with included files filters matching each service's directory
C. Manually select which services to build
D. Split into separate repositories

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Build triggers support `includedFiles` and `ignoredFiles` filters. Configure separate triggers per service with `includedFiles` matching that service's directory (e.g., `services/auth/**`). Only changes to those paths trigger the build. This enables efficient monorepo CI/CD. Building everything (A) wastes resources. Manual selection (C) is error-prone. Splitting repos (D) loses monorepo benefits.

**Key Concept:** [Cloud Build Trigger Filters](https://cloud.google.com/build/docs/automating-builds/create-manage-triggers)
</details>

### Question 7
**Scenario:** A company needs their Cloud Build pipelines to access private resources in their VPC (e.g., internal Artifact Registry, databases for testing). Build workers should not be exposed to the public internet. How should this be configured?

A. Use public Cloud Build workers with firewall rules
B. Configure Cloud Build private pools in a peered VPC with no external IP
C. Use only public registries and services
D. VPN from build workers to VPC

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Build private pools run workers in your VPC (via VPC peering), enabling access to private resources. Workers can be configured without external IPs, restricting all traffic to internal networks. This provides both private access and security. Public workers (A) can't access private VPC resources directly. Public-only services (C) doesn't meet the requirement. VPN (D) is complex and not the native solution.

**Key Concept:** [Cloud Build Private Pools](https://cloud.google.com/build/docs/private-pools/private-pools-overview)
</details>

### Question 8
**Scenario:** A team wants to implement GitOps for their Kubernetes deployments. Application configurations in Git should be automatically synchronized to GKE clusters. Drift from the desired state should be detected and corrected. What GCP feature provides this?

A. Cloud Build triggers deploying on every commit
B. Config Sync with a Git repository as source of truth
C. Scheduled kubectl apply jobs
D. Manual deployment approval for each change

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Config Sync continuously watches a Git repository and synchronizes Kubernetes resources to clusters. It detects drift (manual changes to clusters) and reconciles back to the Git-defined state. This is true GitOps - Git is the source of truth. Cloud Build triggers (A) deploy on commits but don't detect drift. Scheduled jobs (C) introduce delay. Manual approval (D) isn't automated synchronization.

**Key Concept:** [Config Sync](https://cloud.google.com/anthos-config-management/docs/config-sync-overview)
</details>

### Question 9
**Scenario:** A deployment pipeline needs to promote releases through dev → staging → production environments. Each promotion should require approval for production, and the team wants the ability to rollback quickly if issues arise. What service manages this?

A. Custom scripts managing deployments
B. Cloud Deploy with delivery pipeline defining targets, approval requirements, and rollback capability
C. Separate CI/CD pipelines per environment
D. Direct deployment to production

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Deploy delivery pipelines define ordered targets (dev, staging, prod). Approval requirements can be set per target - production requires approval while dev auto-promotes. Releases are immutable, enabling instant rollback to previous releases. The full history is maintained. Custom scripts (A) require building all this functionality. Separate pipelines (C) lose the promotion concept. Direct production (D) is dangerous.

**Key Concept:** [Cloud Deploy Delivery Pipeline](https://cloud.google.com/deploy/docs/overview)
</details>

### Question 10
**Scenario:** A team needs to ensure that code changes are reviewed before being deployed. They want to enforce that all deployments originate from pull requests that have been approved by at least two reviewers. How can this be enforced technically?

A. Document the process and trust developers
B. GitHub branch protection rules requiring 2 approvals + Cloud Build triggers only on merges to protected branches
C. Verbal approval before deployments
D. Post-deployment code review

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** GitHub branch protection rules enforce that PRs to protected branches (e.g., main) require minimum approvals before merge. Cloud Build triggers configured on push to main only run after merge, ensuring only approved code is built and deployed. This creates technical enforcement. Trust and verbal approval (A, C) aren't enforceable. Post-deployment review (D) is too late.

**Key Concept:** [GitHub Branch Protection](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/defining-the-mergeability-of-pull-requests/about-protected-branches)
</details>

### Question 11
**Scenario:** A company wants to implement canary deployments for their Cloud Run services. Traffic should gradually shift from 0% → 5% → 25% → 100% to new revisions, with automatic rollback if error rate exceeds 1%. What approach enables this?

A. Manual traffic splitting in Cloud Run
B. Cloud Deploy with Cloud Run targets using automated canary strategy with Cloud Monitoring metrics for verification
C. All-at-once deployment with monitoring
D. Blue-green deployment only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Deploy supports automated canary deployments for Cloud Run. Define phases (5%, 25%, 100%) with verification using Cloud Monitoring metrics. If error rate exceeds threshold, automatic rollback occurs. This provides automated, metric-driven progressive delivery. Manual splitting (A) requires human intervention at each phase. All-at-once (C) doesn't provide gradual rollout. Blue-green (D) is a different pattern without gradual shifting.

**Key Concept:** [Cloud Deploy Canary Deployments](https://cloud.google.com/deploy/docs/deployment-strategies/canary)
</details>

### Question 12
**Scenario:** A CI pipeline runs integration tests that require a Cloud SQL database. Tests should run in parallel for speed, but each test run needs isolated database state. Creating a new Cloud SQL instance per test is too slow. What approach provides isolation with speed?

A. Share a single test database across all tests
B. Use a persistent test Cloud SQL instance with separate databases/schemas per test run, or transaction rollback isolation
C. Skip integration tests
D. Run tests sequentially

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** A persistent test instance is fast to connect to. Isolation can be achieved through: separate databases per test run (create database is fast), schema per test run, or wrapping each test in a transaction that's rolled back (no cleanup needed). This balances speed and isolation. Shared database (A) causes test interference. Skipping tests (C) reduces confidence. Sequential tests (D) are slow.

**Key Concept:** [Testing Best Practices](https://cloud.google.com/architecture/devops/devops-tech-continuous-testing)
</details>

### Question 13
**Scenario:** A team maintains a shared Terraform module used by multiple application teams. They need to version the module, run automated tests when changes are made, and publish tested versions for consumption. How should this be managed?

A. Copy module code into each consuming repository
B. Store module in a Git repository with tagged versions, CI testing on changes, and consumption via versioned Git references or Artifact Registry
C. Single shared folder accessed by all projects
D. No versioning - always use latest

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Git repository with tags provides versioning. CI/CD tests modules on changes (terraform validate, plan against test infrastructure). Consumers reference specific tags (git::https://...?ref=v1.2.0) or modules published to Artifact Registry. This enables controlled updates and stability. Copying (A) leads to drift. Shared folder (C) doesn't provide isolation or versioning. No versioning (D) causes unexpected breaking changes.

**Key Concept:** [Terraform Module Versioning](https://developer.hashicorp.com/terraform/language/modules/sources)
</details>

### Question 14
**Scenario:** A Cloud Build pipeline builds container images and pushes to Artifact Registry. The team needs to automatically clean up old images to control storage costs while retaining the latest 10 versions and any images currently deployed. How should this be implemented?

A. Manual cleanup periodically
B. Artifact Registry cleanup policies retaining recent versions, with GKE integration to protect deployed images
C. Delete all images after 30 days
D. Unlimited retention

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Artifact Registry supports cleanup policies that delete old images while retaining a specified number of recent versions. Integration with GKE prevents deletion of images currently deployed to clusters. This automates cost management while ensuring availability. Manual cleanup (A) is error-prone and forgotten. Time-based deletion (C) might delete actively used images. Unlimited retention (D) increases costs.

**Key Concept:** [Artifact Registry Cleanup Policies](https://cloud.google.com/artifact-registry/docs/repositories/cleanup-policy)
</details>

### Question 15
**Scenario:** A team needs to store and manage secrets (API keys, database passwords) used by their CI/CD pipelines and applications. Secrets should be versioned, access-controlled, and automatically rotated where possible. What service provides this?

A. Environment variables in Cloud Build configs
B. Secret Manager with IAM access control, versioning, and rotation schedules
C. Encrypted files in Git repository
D. Secrets in application code

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Secret Manager provides centralized secret storage with IAM access control (who can access which secrets), automatic versioning, and rotation scheduling for supported secrets. Cloud Build and Cloud Run can access secrets natively. Environment variables in configs (A) are visible in build logs. Git encrypted files (C) risk exposure and lack rotation. Code (D) is insecure.

**Key Concept:** [Secret Manager](https://cloud.google.com/secret-manager/docs/overview)
</details>

---

## Domain 3: Applying Site Reliability Engineering Practices (Questions 16-26)

### Question 16
**Scenario:** A service team needs to define reliability targets for their customer-facing API. The API handles payment processing, so reliability is critical, but the team also needs to ship features. How should they balance reliability and velocity?

A. Target 100% availability
B. Define SLOs (e.g., 99.9% availability), track error budget, and use error budget policy to balance velocity with reliability work
C. No reliability targets - fix issues as they arise
D. Freeze all feature development until perfect reliability

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** SLOs define acceptable reliability (99.9% = 43 minutes downtime/month). Error budget is the allowed unreliability. When error budget is healthy, ship features (controlled risk). When exhausted, focus on reliability. Error budget policy documents this decision framework. 100% (A) is impossible and stops all progress. No targets (C) leads to crisis-driven operations. Freezing (D) prevents business value delivery.

**Key Concept:** [SLO/Error Budget](https://sre.google/sre-book/service-level-objectives/)
</details>

### Question 17
**Scenario:** A production service experienced an outage due to a configuration change. The team needs to understand what happened, prevent recurrence, and share learnings. What SRE practice addresses this?

A. Blame the person who made the change
B. Conduct a blameless postmortem documenting timeline, root cause, contributing factors, and action items
C. Ignore the incident and move on
D. Immediately fire the engineer involved

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Blameless postmortems focus on systemic factors - what allowed the failure to happen and how to prevent it. Documenting timeline, root causes, contributing factors, and actionable items creates organizational learning. Blame culture (A, D) hides problems and prevents honest discussion. Ignoring (C) misses improvement opportunities and likely leads to recurrence.

**Key Concept:** [Postmortem Culture](https://sre.google/sre-book/postmortem-culture/)
</details>

### Question 18
**Scenario:** A team is measuring their service's availability SLI. They need to define "availability" in a way that reflects user experience. The service is a REST API serving both interactive users and batch processing jobs. How should availability be measured?

A. Server uptime percentage
B. Successful requests / total requests, measured at the load balancer with separate SLIs for interactive (latency-sensitive) and batch (throughput-sensitive) workloads
C. Whether the server process is running
D. Manual user surveys

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Request success rate at the load balancer measures actual user experience - what portion of user requests succeeded. Separate SLIs recognize that interactive users care about latency while batch jobs care about throughput. This enables appropriate targets per workload. Server uptime (A, C) doesn't reflect user-visible failures (overloaded but "up"). Surveys (D) are slow and subjective.

**Key Concept:** [Measuring Availability](https://sre.google/sre-book/monitoring-distributed-systems/)
</details>

### Question 19
**Scenario:** A company wants to reduce the toil involved in their operations. Currently, operators manually check system health dashboards, restart failed services, and scale resources based on load. What SRE practice addresses this?

A. Hire more operators
B. Automate repetitive tasks: health checks via synthetic monitoring, auto-restart via GKE liveness probes, autoscaling via HPA/VPA
C. Reduce service offerings to simplify operations
D. Accept manual operations as necessary

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Toil is manual, repetitive, automatable work that scales with service size. Synthetic monitoring automates health checks. Liveness/readiness probes enable automatic restart of unhealthy containers. Horizontal/Vertical Pod Autoscaling automates scaling decisions. This frees operators for valuable work. More operators (A) scales toil linearly. Reducing services (C) limits business. Accepting toil (D) prevents improvement.

**Key Concept:** [Eliminating Toil](https://sre.google/sre-book/eliminating-toil/)
</details>

### Question 20
**Scenario:** A team manages a critical service with 99.99% SLO (52 minutes downtime/year). They want to validate their incident response procedures and failover mechanisms before actual incidents. How should they test these?

A. Wait for real incidents to test procedures
B. Conduct regular game days/chaos engineering exercises simulating failures in production or staging
C. Read the runbooks without practicing
D. Assume procedures will work when needed

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Game days are planned exercises where teams simulate failures (inject faults) and practice response. This validates runbooks, identifies gaps, builds muscle memory, and tests automated failover. Real incidents (A) risk SLO when discovering gaps. Reading without practice (C) doesn't validate execution. Assumptions (D) lead to surprises during real incidents.

**Key Concept:** [Chaos Engineering](https://sre.google/sre-book/accelerating-sre-on-call/)
</details>

### Question 21
**Scenario:** An on-call engineer receives alerts throughout the night, but investigation shows most alerts don't require action - they're transient issues that self-resolve. This causes alert fatigue. How should this be addressed?

A. Disable all alerting to let engineers sleep
B. Review and tune alert thresholds, implement burn rate alerts instead of instantaneous alerts, suppress transient issues
C. Add more on-call engineers to distribute fatigue
D. Ignore alerts and wait for customer reports

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Alert fatigue reduces response effectiveness. Solutions: tune thresholds based on actual impact, use burn rate/multi-window alerts that fire only for sustained issues, implement alert grouping/deduplication. Alerts should be actionable - if no action is needed, the alert shouldn't fire. Disabling (A) and ignoring (D) miss real issues. More engineers (C) distributes but doesn't solve the problem.

**Key Concept:** [Alerting on SLOs](https://sre.google/workbook/alerting-on-slos/)
</details>

### Question 22
**Scenario:** A service depends on multiple downstream services. When a downstream service fails, the upstream service should degrade gracefully rather than failing entirely. What patterns enable this?

A. No error handling - let failures propagate
B. Circuit breakers, timeouts, fallbacks, and bulkheads
C. Retry infinitely until downstream recovers
D. Tightly couple to downstream services

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Circuit breakers stop calling failing services, preventing cascade failures. Timeouts bound wait time for slow dependencies. Fallbacks provide degraded functionality (cached data, default values). Bulkheads isolate failures to specific components. Together these implement graceful degradation. Propagating failures (A) causes cascades. Infinite retries (C) overwhelm recovering services. Tight coupling (D) makes isolation impossible.

**Key Concept:** [Resilience Patterns](https://sre.google/sre-book/addressing-cascading-failures/)
</details>

### Question 23
**Scenario:** A team needs to safely roll out a significant infrastructure change (migrating from regional to multi-regional database). The change carries risk of data inconsistency or service disruption. What approach minimizes risk?

A. Big bang migration over a weekend
B. Incremental rollout: shadow traffic to new infrastructure, gradual traffic shifting, feature flags for rollback capability
C. No testing - deploy directly to production
D. Announce planned downtime and migrate

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Incremental rollout reduces blast radius. Shadow traffic validates the new system without user impact. Gradual shifting allows monitoring at each stage. Feature flags enable instant rollback. This is progressive delivery for infrastructure. Big bang (A) risks complete failure. No testing (C) is reckless. Planned downtime (D) affects users and doesn't allow incremental validation.

**Key Concept:** [Progressive Rollouts](https://sre.google/workbook/canarying-releases/)
</details>

### Question 24
**Scenario:** A team's service receives traffic from multiple customers. One customer's usage pattern (burst of requests) is impacting other customers' experience. How should this be handled?

A. Rate limit all customers equally
B. Implement per-customer rate limiting and quota management with prioritization for critical customers
C. Ask the problematic customer to reduce usage
D. Scale infinitely to handle all traffic

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Per-customer rate limiting isolates noisy neighbors, preventing one customer from impacting others. Quota management sets usage limits per customer. Priority queuing can favor critical customers during overload. This enables fair resource sharing. Equal limits (A) may be too restrictive for good customers. Requesting reduction (C) isn't enforceable. Infinite scaling (D) is cost-prohibitive and may not be fast enough.

**Key Concept:** [Rate Limiting](https://sre.google/sre-book/handling-overload/)
</details>

### Question 25
**Scenario:** During an incident, multiple teams need to coordinate response. Communication is chaotic - people are talking over each other, status updates are unclear, and decisions are delayed. What SRE practice improves incident coordination?

A. Let everyone contribute equally without structure
B. Implement incident command system with defined roles (Incident Commander, Communications Lead, Operations Lead)
C. Single person handles everything
D. No coordination - each team works independently

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Incident command system provides clear roles: Incident Commander makes decisions, Communications Lead handles stakeholder updates, Operations Lead coordinates technical work. This prevents confusion and enables parallel work streams with clear accountability. Unstructured contribution (A) causes chaos. Single person (C) creates bottleneck. Independent work (D) leads to conflicting actions.

**Key Concept:** [Incident Response](https://sre.google/sre-book/managing-incidents/)
</details>

### Question 26
**Scenario:** A team is evaluating whether to add a new feature that requires an additional external dependency. From an SRE perspective, what concerns should be raised?

A. No concerns - add dependencies freely
B. Assess: dependency's reliability (does it have SLOs?), failure modes, fallback options, and impact on service's overall reliability budget
C. Reject all new dependencies
D. Only consider feature functionality

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** External dependencies impact service reliability. If the dependency has 99% availability and your target is 99.9%, the dependency alone could consume your error budget. Assess: Can you cache/fallback? What happens when it fails? Does the vendor publish SLOs? This informs architectural decisions. Free addition (A) risks reliability. Rejecting all (C) prevents useful integrations. Functionality only (D) ignores operational concerns.

**Key Concept:** [Dependency Management](https://sre.google/sre-book/service-level-objectives/)
</details>

---

## Domain 4: Implementing Service Monitoring Strategies (Questions 27-35)

### Question 27
**Scenario:** A team needs to monitor their GKE application's health. They want to know when pods are unhealthy, when resource utilization is high, and when error rates increase. What monitoring configuration provides this visibility?

A. Only check if nodes are running
B. GKE dashboard with pod health, Cloud Monitoring with resource metrics (CPU, memory), custom metrics for error rates, and alerts on thresholds
C. Application logs only
D. Manual health checks

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** GKE dashboard shows pod/node status. Cloud Monitoring collects system metrics (CPU, memory, network) automatically. Custom metrics (via OpenTelemetry or Prometheus) capture application-specific data like error rates. Alerting policies notify on threshold breaches. Node status only (A) misses pod-level issues. Logs (C) require parsing for metrics. Manual checks (D) don't scale.

**Key Concept:** [GKE Monitoring](https://cloud.google.com/kubernetes-engine/docs/concepts/dashboards)
</details>

### Question 28
**Scenario:** A service has defined an SLO of 99.9% availability measured by successful HTTP responses. The team wants alerts that fire when they're in danger of missing the SLO, not on every error. What alerting approach provides this?

A. Alert on every 5xx error
B. Multi-window, multi-burn-rate alerts based on error budget consumption rate
C. Alert when availability drops below 99.9%
D. No alerting - check dashboards manually

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Multi-burn-rate alerts measure how quickly error budget is being consumed. Fast burn (consuming budget quickly) triggers immediate alerts. Slow burn triggers less urgent alerts. Multi-window (1h, 6h) prevents alerting on brief spikes. This focuses alerts on SLO risk. Every error (A) causes alert fatigue. Below-threshold alerts (C) fire too late. Manual checks (D) miss issues.

**Key Concept:** [Alerting on SLOs](https://sre.google/workbook/alerting-on-slos/)
</details>

### Question 29
**Scenario:** A team needs to understand the latency distribution of their API, not just the average. Some requests are much slower than others, affecting user experience. What metrics approach captures this?

A. Monitor average latency only
B. Capture latency percentiles (p50, p90, p95, p99) using Cloud Monitoring distribution metrics
C. Count total requests
D. Monitor maximum latency only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Percentiles reveal latency distribution - p50 is median experience, p99 shows worst 1% experience. This identifies long-tail latency affecting some users. Cloud Monitoring distribution metrics capture percentiles automatically for HTTP latency. Averages (A) hide outliers. Total requests (C) don't measure performance. Maximum (D) may be anomalies, not representative.

**Key Concept:** [Distribution Metrics](https://cloud.google.com/monitoring/api/v3/kinds-and-types#metric-kinds)
</details>

### Question 30
**Scenario:** A microservices application has requests flowing through multiple services. When a request is slow, the team needs to identify which service is the bottleneck. What observability tool provides this insight?

A. Aggregate latency metrics per service
B. Cloud Trace providing distributed tracing showing latency breakdown across service calls
C. CPU utilization per service
D. Log timestamps

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Distributed tracing follows individual requests across services, showing exactly how time is spent at each hop. Cloud Trace visualizes this as trace waterfalls, immediately identifying bottleneck services. Aggregate metrics (A) show averages, not individual request paths. CPU (C) may not correlate with specific requests. Log timestamps (D) require manual correlation.

**Key Concept:** [Cloud Trace](https://cloud.google.com/trace/docs/overview)
</details>

### Question 31
**Scenario:** A team wants to detect anomalous behavior in their metrics - sudden drops in traffic, unusual error patterns, or unexpected resource consumption - without manually setting thresholds for every metric. What approach enables this?

A. Set static thresholds for all metrics
B. Use Cloud Monitoring anomaly detection / ML-based alerting policies
C. Only alert on absolute values
D. No anomaly detection - rely on manual observation

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Monitoring supports anomaly detection that learns normal patterns and alerts on deviations. This catches issues without manually tuning every threshold. It adapts to changing baselines (e.g., different weekend traffic). Static thresholds (A) require maintenance and may miss pattern changes. Absolute values (C) don't account for variability. Manual observation (D) doesn't scale.

**Key Concept:** [Anomaly Detection in Cloud Monitoring](https://cloud.google.com/monitoring/alerts/concepts-indepth#anomaly-detection)
</details>

### Question 32
**Scenario:** A team needs to monitor user journeys through their web application (login → search → purchase). They want to know if any step has significantly higher failure rates than others. What monitoring approach provides this business-level visibility?

A. Infrastructure metrics only
B. Synthetic monitoring / uptime checks for critical user flows with Cloud Monitoring
C. Server-side logging only
D. Monthly user surveys

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Synthetic monitoring (uptime checks) can execute multi-step user journeys, measuring success/failure and latency per step. This provides continuous validation of user flows from external perspective. Infrastructure metrics (A) don't reflect user experience. Server logs (C) miss client-side issues. Surveys (D) are delayed and subjective.

**Key Concept:** [Uptime Checks](https://cloud.google.com/monitoring/uptime-checks)
</details>

### Question 33
**Scenario:** A team operates multiple services across multiple projects. They need a unified view of health across all services with the ability to drill down into specific issues. How should they organize their monitoring?

A. Separate dashboards per project with no aggregation
B. Cloud Monitoring workspace scoping multiple projects with service-level dashboards and aggregate views
C. Single giant dashboard with all metrics
D. No dashboards - use alerting only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Monitoring workspaces can scope multiple projects, providing cross-project visibility. Service-level dashboards show per-service health. Aggregate dashboards provide overview. This enables both summary views and detailed drill-down. Separate dashboards (A) lack unified view. Single dashboard (C) is overwhelming. Alerting only (D) lacks proactive visibility and context.

**Key Concept:** [Cloud Monitoring Workspaces](https://cloud.google.com/monitoring/workspaces)
</details>

### Question 34
**Scenario:** A team needs to correlate metrics, logs, and traces when investigating issues. Currently these are separate systems requiring context-switching. How can they improve the investigation experience?

A. Use separate tools and manually correlate
B. Enable Cloud Trace and Cloud Logging integration with trace ID propagation, link from metrics to traces to logs
C. Only use one signal type
D. Don't investigate - just restart services

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Trace ID propagation links traces and logs - clicking a trace shows associated log entries, and logs can link to parent traces. Metrics can link to exemplars (specific traces representing the metric). This enables seamless navigation between observability signals. Manual correlation (A) is slow. Single signal (C) loses context. Blind restart (D) doesn't resolve root causes.

**Key Concept:** [Correlating Telemetry](https://cloud.google.com/architecture/devops/devops-measurement-monitoring-and-observability)
</details>

### Question 35
**Scenario:** A team manages a service with dependencies on other team's services. When their service receives errors from a dependency, they're often alerted before the owning team knows. How can they improve this cross-team situation?

A. Alert their team on dependency failures
B. Implement shared SLOs with dependency teams visible in shared dashboards, with dependency teams responsible for their alerts
C. Ignore dependency failures
D. Own all dependencies themselves

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Shared SLOs create accountability - dependency teams define and monitor their own SLOs. Shared dashboards provide visibility across team boundaries. Dependency teams alert on their own issues. This distributes responsibility appropriately. Alerting on others' failures (A) creates noise without ownership. Ignoring (C) misses issues affecting users. Owning all (D) doesn't scale organizationally.

**Key Concept:** [SLOs Across Teams](https://sre.google/sre-book/service-level-objectives/)
</details>

---

## Domain 5: Optimizing Service Performance (Questions 36-40)

### Question 36
**Scenario:** A web application has slow page load times. Analysis shows that the backend API is fast, but there's significant latency for users far from the servers. Most content is static. How can performance be improved for global users?

A. Add more servers in the same region
B. Deploy Cloud CDN to cache static content at edge locations globally
C. Optimize database queries
D. Increase server CPU

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud CDN caches content at Google's edge locations worldwide, reducing latency for users by serving content from nearby locations. Since content is mostly static, cache hit rates will be high. More same-region servers (A) don't reduce geographic latency. Database optimization (C) is irrelevant if API is already fast. CPU (D) doesn't address network latency.

**Key Concept:** [Cloud CDN](https://cloud.google.com/cdn/docs/overview)
</details>

### Question 37
**Scenario:** A Cloud Run service experiences high latency during traffic spikes due to cold starts as new instances spin up. The service is latency-sensitive for user requests. How can cold start impact be reduced?

A. Accept cold start latency
B. Configure minimum instances > 0 to keep warm instances ready
C. Migrate to Compute Engine
D. Reduce service traffic

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cloud Run minimum instances setting keeps specified instances warm and ready, eliminating cold starts up to that concurrency. For latency-sensitive services, setting minimum instances based on baseline traffic ensures instant response. The tradeoff is cost for idle instances. Accepting latency (A) impacts users. Compute Engine (C) loses serverless benefits. Reducing traffic (D) isn't practical.

**Key Concept:** [Cloud Run Min Instances](https://cloud.google.com/run/docs/configuring/min-instances)
</details>

### Question 38
**Scenario:** A GKE application is experiencing memory pressure on nodes, leading to pod evictions and service disruption. Memory usage varies significantly between pods of the same deployment. How can resource allocation be optimized?

A. Increase memory for all pods uniformly
B. Implement Vertical Pod Autoscaler (VPA) to right-size pod resource requests based on actual usage
C. Add more nodes
D. Remove memory limits

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** VPA analyzes actual resource usage and adjusts pod resource requests/limits accordingly. This right-sizes allocation based on real needs rather than guesses. Pods that need more memory get it; those that don't free resources. Uniform increase (A) wastes resources on light pods. More nodes (C) don't fix inefficient allocation. Removing limits (D) risks node exhaustion from unbounded memory use.

**Key Concept:** [Vertical Pod Autoscaler](https://cloud.google.com/kubernetes-engine/docs/concepts/verticalpodautoscaler)
</details>

### Question 39
**Scenario:** A team's application makes frequent calls to Cloud Storage to retrieve configuration files. Each request has latency overhead even though the files rarely change. How can this be optimized?

A. Accept the latency as necessary
B. Implement caching using Memorystore for Redis with TTL-based invalidation
C. Store configuration in application code
D. Read configuration only at startup

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Memorystore for Redis provides sub-millisecond caching. Configuration files read from Cloud Storage are cached with TTL for automatic refresh. This reduces storage API calls and latency while keeping configuration reasonably current. Accepting latency (A) impacts performance unnecessarily. Code embedding (C) requires redeployment for changes. Startup-only (D) requires restart for updates.

**Key Concept:** [Memorystore for Redis](https://cloud.google.com/memorystore/docs/redis/redis-overview)
</details>

### Question 40
**Scenario:** Cloud Profiler shows that a production application spends significant CPU time in JSON serialization. The application processes millions of requests daily on Cloud Run. How can this performance bottleneck be addressed?

A. Increase CPU allocation
B. Switch to a more efficient serialization format (Protocol Buffers, MessagePack) for internal service communication
C. Reduce traffic
D. Cache serialized responses

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Binary serialization formats like Protocol Buffers or MessagePack are significantly faster than JSON and produce smaller payloads. For internal service communication (not user-facing APIs), this reduces CPU time and network transfer. Caching (D) helps for repeated responses. More CPU (A) addresses symptom not cause. Reducing traffic (C) isn't practical. Caching serialized responses (D) works but requires cache management.

**Key Concept:** [Cloud Profiler](https://cloud.google.com/profiler/docs/about-profiler)
</details>
