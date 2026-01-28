# Azure DevOps Engineer Expert (AZ-400) Practice Questions

40 scenario-based questions weighted by exam domain.

## Exam Domain Breakdown
| Domain | Weight | Questions |
|--------|--------|-----------|
| Configure processes and communications | 10-15% | 5 |
| Design and implement source control | 15-20% | 7 |
| Design and implement build and release pipelines | 40-45% | 17 |
| Develop a security and compliance plan | 10-15% | 5 |
| Implement an instrumentation strategy | 10-15% | 6 |

---

## Domain 1: Configure Processes and Communications (Questions 1-5)

### Question 1
**Scenario:** A development team wants to implement agile project management in Azure DevOps. They need to track user stories, tasks, bugs, and sprints. They also want dashboards showing sprint progress and team velocity. What should they configure?

A. Use only Git branches for tracking work
B. Azure Boards with Scrum process template, sprint configuration, and dashboard widgets
C. Third-party project management tool
D. SharePoint lists

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Boards provides native agile work item tracking. Scrum process template includes user stories, tasks, bugs, and sprint features. Built-in dashboard widgets show burndown charts, velocity, and sprint progress. Git branches (A) don't track work items. Third-party (C) requires integration. SharePoint (D) isn't designed for agile.

**Key Concept:** [Azure Boards](https://learn.microsoft.com/en-us/azure/devops/boards/get-started/what-is-azure-boards)
</details>

### Question 2
**Scenario:** A company needs to notify the team automatically when a build fails, a work item is assigned, or a pull request is created. Notifications should go to Microsoft Teams. What should they configure?

A. Manual email notifications
B. Azure DevOps Service Hooks with Microsoft Teams integration
C. Check Azure DevOps portal manually
D. Email subscriptions only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Service Hooks provide event-driven integration with external services. Teams integration posts notifications for builds, work items, PRs, and more to specified channels. Automatic and real-time. Manual checks (A, C) aren't proactive. Email subscriptions (D) work but Teams provides better collaboration.

**Key Concept:** [Service Hooks](https://learn.microsoft.com/en-us/azure/devops/service-hooks/overview)
</details>

### Question 3
**Scenario:** A team has multiple projects with shared requirements for work item templates, process customization, and team settings. They want to define these once and inherit them. What should they configure?

A. Configure each project separately
B. Create a custom inherited process and apply it to projects
C. Use Excel for work items
D. Copy settings manually between projects

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Inherited processes in Azure DevOps allow customizing work item types, states, fields, and rules. The custom process applies to multiple projects, ensuring consistency. Changes propagate to all projects using the process. Per-project configuration (A, D) causes drift. Excel (C) doesn't define processes.

**Key Concept:** [Process Customization](https://learn.microsoft.com/en-us/azure/devops/organizations/settings/work/inheritance-process-model)
</details>

### Question 4
**Scenario:** A team wants to automate the process of creating a work item when a deployment fails. The work item should include failure details and be assigned to the on-call engineer. What should they implement?

A. Manual work item creation
B. Pipeline task that creates work item on failure using REST API or built-in task
C. Email notification only
D. Log the failure only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure Pipelines can conditionally run tasks on failure (condition: failed()). The CreateWorkItem task or REST API creates work items with dynamic values from pipeline variables. Assignment can be parameterized. Manual creation (A) is slow. Email (C) doesn't create trackable work items. Logging only (D) isn't actionable.

**Key Concept:** [Pipeline Conditions](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/conditions)
</details>

### Question 5
**Scenario:** A company has developers across multiple time zones. They want to maintain documentation about their DevOps practices, architecture decisions, and runbooks in a collaborative, version-controlled manner. What should they use?

A. SharePoint documents
B. Azure DevOps Wiki with Git-backed content
C. Confluence
D. Email chains

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure DevOps Wiki can be Git-backed (stored as Markdown in a repo), providing version control, pull request workflows, and integration with Azure DevOps. Developers can collaborate using familiar Git workflows. SharePoint (A) lacks version control integration. Confluence (C) is external. Email (D) isn't documentation.

**Key Concept:** [Azure DevOps Wiki](https://learn.microsoft.com/en-us/azure/devops/project/wiki/wiki-create-repo)
</details>

---

## Domain 2: Design and Implement Source Control (Questions 6-12)

### Question 6
**Scenario:** A team wants to enforce that all code changes go through pull requests with at least two reviewers, automated builds must pass, and work items must be linked. What should they configure?

A. Verbal policy only
B. Branch policies on the main branch with required reviewers, build validation, and linked work items
C. Post-commit reviews
D. Lock the branch

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Branch policies enforce PR requirements: minimum reviewers, build validation (PR builds must pass), linked work items, and more. Policies prevent direct pushes to protected branches. Verbal policy (A) isn't enforced. Post-commit reviews (C) are after the fact. Locking (D) prevents all changes.

**Key Concept:** [Branch Policies](https://learn.microsoft.com/en-us/azure/devops/repos/git/branch-policies)
</details>

### Question 7
**Scenario:** A team uses Git in Azure Repos. They want to prevent force pushes to the main branch and ensure the branch cannot be deleted. What should they configure?

A. Branch permissions to deny force push and delete for contributors
B. Git hooks only
C. Repository settings for the entire repo
D. Client-side Git configuration

<details>
<summary>Answer</summary>

**Correct: A**

**Why:** Branch security/permissions in Azure Repos allow granular control per branch. You can deny "Force push" and "Remove others' locks" to prevent rewriting history, and deny "Delete" to prevent branch deletion. Git hooks (B) are per-client. Repo settings (C) are repo-wide. Client config (D) isn't enforced server-side.

**Key Concept:** [Branch Permissions](https://learn.microsoft.com/en-us/azure/devops/repos/git/branch-permissions)
</details>

### Question 8
**Scenario:** A large development team wants to implement a branching strategy that supports continuous delivery with short-lived feature branches, release branches for production stabilization, and hotfixes. What strategy should they use?

A. Single branch for everything
B. Git Flow with feature, develop, release, main, and hotfix branches
C. Feature branches only
D. Fork-based workflow for all developers

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Git Flow provides structured branching: feature branches for development, develop branch for integration, release branches for stabilization, main for production, and hotfix branches for urgent fixes. Supports continuous delivery with clear paths to production. Single branch (A) is too simple. Feature only (C) lacks release management. Forks (D) are for external contributors.

**Key Concept:** [Git Flow](https://learn.microsoft.com/en-us/azure/devops/repos/git/git-branching-guidance)
</details>

### Question 9
**Scenario:** A developer accidentally committed a password to a Git repository. They need to remove it from the entire history. What should they do?

A. Delete the file in a new commit
B. Use git filter-repo or BFG Repo-Cleaner to rewrite history, then force push
C. Create a new repository
D. Ignore it since it's in history

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** git filter-repo or BFG Repo-Cleaner can remove sensitive data from all commits in history. After rewriting, force push updates the remote. All developers must re-clone. Also rotate the exposed credential. Simple deletion (A) leaves it in history. New repo (C) loses history. Ignoring (D) is a security risk.

**Key Concept:** [Remove Sensitive Data](https://learn.microsoft.com/en-us/azure/devops/repos/git/remove-binaries)
</details>

### Question 10
**Scenario:** A team wants to implement code review comments that persist across PR updates and allow reviewers to see if their comments have been addressed. What Azure DevOps feature supports this?

A. Email comments
B. PR comment threads with resolution status
C. Work item comments
D. Wiki pages

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** PR comment threads in Azure DevOps persist across updates. Comments can be marked as resolved, active, or won't fix. Reviewers see pending comments even after new pushes. Policies can require all comments resolved before merge. Email (A) isn't tracked. Work items (C) are separate from code review. Wiki (D) is documentation.

**Key Concept:** [PR Reviews](https://learn.microsoft.com/en-us/azure/devops/repos/git/pull-requests)
</details>

### Question 11
**Scenario:** A company acquires another company using a different Git platform. They need to migrate 50 repositories to Azure Repos while preserving complete history and branches. What approach should they use?

A. Copy only the latest code
B. Import repositories using Azure DevOps import feature or git push --mirror
C. Recreate repositories manually
D. Use file copy

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure DevOps Import can clone from Git URLs. Alternatively, git clone --mirror followed by git push --mirror preserves all branches, tags, and history. Both maintain complete repository history. Latest code only (A) loses history. Manual recreation (C) and file copy (D) lose Git history.

**Key Concept:** [Import Repository](https://learn.microsoft.com/en-us/azure/devops/repos/git/import-git-repository)
</details>

### Question 12
**Scenario:** A team uses Git submodules to share common libraries across multiple repositories. Builds fail because submodules aren't being cloned. What pipeline configuration is needed?

A. Clone submodules in a script step
B. Set checkout: submodules: true (or recursive) in the pipeline YAML
C. Manually maintain copies of shared code
D. Use package manager instead

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Pipeline checkout step supports submodules option: true (one level) or recursive (nested submodules). This clones submodules automatically. Script step (A) works but is more complex. Manual copies (C) defeat the purpose. Package managers (D) are an alternative architecture but don't solve the submodule issue.

**Key Concept:** [Checkout Submodules](https://learn.microsoft.com/en-us/azure/devops/pipelines/repos/pipeline-options-for-git)
</details>

---

## Domain 3: Design and Implement Build and Release Pipelines (Questions 13-29)

### Question 13
**Scenario:** A developer needs to create a CI pipeline that builds on every push to any branch, runs tests, and publishes artifacts. The pipeline definition should be version-controlled with the code. What should they create?

A. Classic build pipeline in the UI
B. YAML pipeline (azure-pipelines.yml) in the repository root
C. Jenkins pipeline
D. Manual builds

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** YAML pipelines are defined in code, version-controlled with the repository. Pipeline as code allows reviews, history, and branching. Triggers can be configured for any branch. Classic pipelines (A) are UI-based, not version-controlled. Jenkins (C) is a different tool. Manual builds (D) aren't CI.

**Key Concept:** [YAML Pipelines](https://learn.microsoft.com/en-us/azure/devops/pipelines/yaml-schema)
</details>

### Question 14
**Scenario:** A pipeline needs to deploy to development automatically but require manual approval before deploying to production. How should this be configured?

A. Two separate pipelines
B. Multi-stage YAML pipeline with environment approval on the production environment
C. Manual trigger for production
D. Delay task in the pipeline

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Multi-stage YAML pipelines define stages (build, dev, prod). Environments in Azure DevOps can have approval checks. Production environment requires approval before the stage runs. Single pipeline with promotion. Separate pipelines (A) duplicate configuration. Manual trigger (C) requires separate action. Delay (D) isn't approval.

**Key Concept:** [Environment Approvals](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/approvals)
</details>

### Question 15
**Scenario:** A team wants to implement infrastructure as code for their Azure resources. The pipeline should deploy ARM templates or Bicep files with parameterized values per environment. What task should they use?

A. PowerShell script with az CLI
B. AzureResourceManagerTemplateDeployment task with parameter files per environment
C. Manual portal deployment
D. Azure CLI task only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** AzureResourceManagerTemplateDeployment task is designed for ARM/Bicep deployments. It supports template files, parameter files (dev.parameters.json, prod.parameters.json), and service connections for authentication. Parameter files enable environment-specific values. Scripts (A, D) work but require more code. Portal (C) isn't IaC.

**Key Concept:** [ARM Deployment Task](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/azure-resource-manager-template-deployment-v3)
</details>

### Question 16
**Scenario:** A pipeline builds a .NET application. The build takes 15 minutes, with 10 minutes spent restoring NuGet packages. How can this be optimized?

A. Use faster agents
B. Implement pipeline caching for NuGet packages
C. Remove dependencies
D. Skip package restore

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Pipeline caching stores NuGet packages between runs. Subsequent builds use cached packages instead of downloading. Cache key based on packages.lock.json ensures cache invalidation when dependencies change. Faster agents (A) don't reduce download time. Removing dependencies (C) may break the app. Skipping restore (D) causes build failures.

**Key Concept:** [Pipeline Caching](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/caching)
</details>

### Question 17
**Scenario:** A team has common pipeline code used across 20 repositories (build steps, deployment steps). They want to maintain this once and reference it from all pipelines. What should they implement?

A. Copy YAML to each repository
B. YAML templates in a central repository, referenced with resources
C. Classic task groups
D. Separate pipeline for common steps

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** YAML templates can be stored in a central repository. Other pipelines reference them using resources.repositories and template references. Changes to templates propagate to all consuming pipelines. Copying (A) causes drift. Classic task groups (C) aren't YAML. Separate pipelines (D) add complexity.

**Key Concept:** [Template References](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/templates)
</details>

### Question 18
**Scenario:** A pipeline needs to deploy a containerized application to Azure Kubernetes Service. The image should be built, pushed to Azure Container Registry, and deployed using Helm charts. What tasks are needed?

A. Build manually and deploy via kubectl
B. Docker@2 to build/push, HelmDeploy@0 for deployment with ACR authentication
C. Use App Service instead
D. Script everything

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Docker@2 task builds and pushes images to ACR. HelmDeploy@0 packages and deploys Helm charts to AKS with proper authentication via Azure service connection. Tasks handle authentication and configuration. Manual kubectl (A) requires credential management. App Service (C) changes architecture. Scripts (D) require more maintenance.

**Key Concept:** [Deploy to AKS](https://learn.microsoft.com/en-us/azure/devops/pipelines/ecosystems/kubernetes/deploy)
</details>

### Question 19
**Scenario:** A team wants to implement blue-green deployments for their Azure App Service. The pipeline should deploy to a staging slot, run tests, then swap with production. If tests fail, no swap occurs. How should this be configured?

A. Deploy directly to production
B. AzureWebApp task to staging slot, test stage, AzureAppServiceManage task to swap slots
C. Manual slot management
D. Two separate App Services

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Deploy to staging slot first. Test stage runs against staging URL. If tests pass, AzureAppServiceManage task with action: swap swaps staging and production. If tests fail, pipeline stops before swap. Zero-downtime deployment. Direct production (A) risks downtime. Manual swap (C) isn't automated. Separate services (D) require traffic management.

**Key Concept:** [Slot Deployments](https://learn.microsoft.com/en-us/azure/devops/pipelines/targets/webapp)
</details>

### Question 20
**Scenario:** A pipeline needs to run different steps based on the branch being built. Feature branches should only build and test, while main branch should also deploy. How should this be implemented?

A. Separate pipelines for each branch
B. Conditional stages/jobs using condition with Build.SourceBranch variable
C. Manual selection at runtime
D. Always run all steps

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Conditions in YAML evaluate variables like Build.SourceBranch. Deploy stage condition: eq(variables['Build.SourceBranch'], 'refs/heads/main') runs only for main branch. Single pipeline handles all branches. Separate pipelines (A) duplicate build configuration. Manual selection (C) isn't automatic. All steps (D) is unnecessary.

**Key Concept:** [Pipeline Conditions](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/conditions)
</details>

### Question 21
**Scenario:** A security team requires that all production deployments use specific, audited images. Developers shouldn't be able to modify deployment scripts. How should the pipeline be secured?

A. Trust developers completely
B. Use YAML templates with required template checks on environments
C. Manual deployment by security team
D. Classic release pipeline only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Required template checks on environments enforce that pipelines must extend from approved templates. Templates are in a protected repository developers can't modify. Any pipeline deploying to production must use the approved template. Complete trust (A) doesn't meet security requirements. Manual (C) doesn't scale. Classic (D) has similar concerns.

**Key Concept:** [Required Templates](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/environments)
</details>

### Question 22
**Scenario:** A developer needs to securely use an API key in a pipeline without exposing it in the YAML file. The key should be masked in logs. What should they use?

A. Plain text in YAML
B. Azure DevOps pipeline variable marked as secret, or Azure Key Vault integration
C. Environment variable in agent
D. Commit the key to the repository

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Secret variables in Azure DevOps are encrypted and masked in logs. Azure Key Vault variable groups provide additional security with access policies and audit logging. Both prevent exposure in YAML and logs. Plain text (A, D) exposes secrets. Environment variables (C) aren't managed.

**Key Concept:** [Secret Variables](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/variables#secret-variables)
</details>

### Question 23
**Scenario:** A pipeline needs to run integration tests that require a database. The database should be created for the test, used during testing, and destroyed after. What approach should they use?

A. Use production database
B. Docker container for database in pipeline using Docker@2 or container jobs
C. Manual database setup
D. Skip integration tests

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Docker containers in pipelines provide isolated, ephemeral infrastructure. Container jobs or Docker@2 task can start a database container (SQL Server, PostgreSQL) for testing. Container is destroyed when pipeline ends. Production database (A) risks data issues. Manual setup (C) isn't repeatable. Skipping tests (D) reduces quality.

**Key Concept:** [Container Jobs](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/container-phases)
</details>

### Question 24
**Scenario:** A team wants to implement canary deployments to AKS. They want to deploy to 10% of pods first, monitor for errors, then roll out to 100% if successful. What should they use?

A. Manual pod management
B. Kubernetes Deployment strategy with canary manifests and pipeline gates
C. All-at-once deployment
D. Blue-green only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Kubernetes deployments can use canary strategies with separate deployments and service traffic splitting. Pipeline gates using Azure Monitor queries can validate canary health before proceeding. SMI TrafficSplit or Istio enables traffic management. Manual management (A) is error-prone. All-at-once (C) is risky. Blue-green (D) is different pattern.

**Key Concept:** [Canary Deployments](https://learn.microsoft.com/en-us/azure/devops/pipelines/ecosystems/kubernetes/canary-demo)
</details>

### Question 25
**Scenario:** A pipeline runs on Microsoft-hosted agents but needs specific software that isn't pre-installed. Installing the software in each run adds 5 minutes. What's the best solution?

A. Accept the 5-minute overhead
B. Use self-hosted agents with pre-installed software or create custom agent images
C. Remove the software dependency
D. Install once and hope it persists

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Self-hosted agents can have custom software pre-installed. For scale, create custom agent images (VM images or container images) with required software. Agents start ready to build. Overhead (A) wastes time. Removing dependencies (C) may not be possible. Microsoft-hosted agents are ephemeral so software doesn't persist (D).

**Key Concept:** [Self-hosted Agents](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/agents)
</details>

### Question 26
**Scenario:** A company has multiple test environments (QA, UAT, staging). Deployments should progress through each environment sequentially with testing at each stage. How should this be modeled?

A. Single stage with sequential tasks
B. Multi-stage pipeline with stage dependencies (dependsOn) and environment approvals
C. Separate pipelines for each environment
D. Manual deployments

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Multi-stage pipelines model environment progression. Each stage has dependsOn specifying prerequisites (build → QA → UAT → staging). Environments provide approvals and gates between stages. Single pipeline maintains context and artifacts. Single stage (A) can't model dependencies properly. Separate pipelines (C) lose artifact continuity.

**Key Concept:** [Multi-stage Pipelines](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/stages)
</details>

### Question 27
**Scenario:** A deployment pipeline needs to update Azure SQL database schema as part of the deployment. What approach should they use?

A. Manual SQL scripts
B. DACPAC deployment with SqlAzureDacpacDeployment task or EF Core migrations
C. Update schema in production directly
D. Skip schema updates

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** DACPAC (Data-tier Application Package) represents database schema. SqlAzureDacpacDeployment task compares DACPAC to target and generates/applies migration scripts. EF Core migrations is an alternative for .NET projects. Both integrate with pipelines. Manual scripts (A) are error-prone. Direct updates (C) skip testing. Skipping (D) causes issues.

**Key Concept:** [SQL Database Deployment](https://learn.microsoft.com/en-us/azure/devops/pipelines/targets/azure-sqldb)
</details>

### Question 28
**Scenario:** A team wants to implement GitOps for Kubernetes deployments. Configuration changes in Git should automatically sync to the cluster without pipeline triggers. What should they implement?

A. Pipeline triggered on every Git change
B. Flux or ArgoCD GitOps operator in the cluster with Azure DevOps repo as source
C. Manual kubectl apply
D. Direct cluster modifications

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** GitOps operators (Flux, ArgoCD) run in the cluster and continuously reconcile cluster state with Git repository. Changes to Git automatically sync without pipeline triggers. True GitOps pattern. Pipeline triggers (A) are push-based, not pull-based GitOps. Manual apply (C) and direct modifications (D) aren't GitOps.

**Key Concept:** [GitOps with Flux](https://learn.microsoft.com/en-us/azure/azure-arc/kubernetes/conceptual-gitops-flux2)
</details>

### Question 29
**Scenario:** A pipeline deploys to multiple Azure regions in parallel to reduce deployment time. What YAML construct should they use?

A. Sequential stages
B. Jobs within a stage with no dependencies (parallel execution)
C. Matrix strategy
D. Multiple pipelines

<details>
<summary>Answer</summary>

**Correct: C**

**Why:** Matrix strategy creates multiple job instances with different variable values (regions). Jobs run in parallel by default. Single definition with matrix simplifies maintenance. Sequential stages (A) don't run in parallel. Jobs without dependencies (B) work but matrix is cleaner. Multiple pipelines (D) duplicate configuration.

**Key Concept:** [Matrix Strategy](https://learn.microsoft.com/en-us/azure/devops/pipelines/yaml-schema/jobs-job-strategy)
</details>

---

## Domain 4: Develop a Security and Compliance Plan (Questions 30-34)

### Question 30
**Scenario:** A security team needs to scan code for vulnerabilities and license compliance as part of the CI pipeline. They want to block builds that have critical vulnerabilities. What should they implement?

A. Manual code review only
B. GitHub Advanced Security for Azure DevOps or third-party tools like SonarQube, Snyk
C. No security scanning
D. Scan only in production

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** GitHub Advanced Security (GHAzDO) provides secret scanning, code scanning (CodeQL), and dependency scanning. Third-party tools like SonarQube, Snyk, or WhiteSource also integrate with pipelines. Build policies can fail on critical findings. Manual review (A) misses automated detection. No scanning (C) is risky. Production scanning (D) is too late.

**Key Concept:** [GitHub Advanced Security for Azure DevOps](https://learn.microsoft.com/en-us/azure/devops/repos/security/configure-github-advanced-security-features)
</details>

### Question 31
**Scenario:** A company needs to ensure that container images deployed to production have been scanned for vulnerabilities and come from approved registries. What should they implement?

A. Trust all images
B. Azure Container Registry tasks for scanning, Azure Policy for allowed registries
C. Manual image review
D. Use latest tags always

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** ACR vulnerability scanning (with Microsoft Defender for Containers) scans images in the registry. Azure Policy with allowed registries constraint prevents deployment from unapproved sources. Together they enforce image security. Trusting all (A) is insecure. Manual review (C) doesn't scale. Latest tags (D) aren't controlled.

**Key Concept:** [Container Security](https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-containers-introduction)
</details>

### Question 32
**Scenario:** A pipeline uses a service principal to deploy to Azure. The security team wants to limit what the service principal can do and track all its activities. What should they configure?

A. Use global administrator service principal
B. Least-privilege custom RBAC role for the service principal and Azure AD audit logs
C. Shared credentials
D. No access controls

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Custom RBAC role with only necessary permissions (e.g., deploy to specific resource group) limits blast radius. Azure AD audit logs and Activity logs track all API calls by the service principal. Global admin (A) is excessive. Shared credentials (C) lack accountability. No controls (D) is insecure.

**Key Concept:** [Service Principal Security](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/connect-to-azure)
</details>

### Question 33
**Scenario:** A company needs to manage secrets used by pipelines across multiple projects. Secrets should be stored securely, access should be audited, and rotation should be possible without updating pipelines. What should they use?

A. Secret variables in each pipeline
B. Azure Key Vault linked as variable group across projects
C. Plain text configuration
D. Environment variables on agents

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Key Vault variable groups link pipeline variables to Key Vault secrets. Secrets are stored securely with access policies and audit logs. Rotation in Key Vault automatically updates pipeline access (no pipeline changes). Cross-project sharing through variable groups. Per-pipeline secrets (A) don't share or audit centrally.

**Key Concept:** [Key Vault Variable Groups](https://learn.microsoft.com/en-us/azure/devops/pipelines/library/variable-groups#link-secrets-from-an-azure-key-vault)
</details>

### Question 34
**Scenario:** A compliance requirement mandates that all deployments are traceable to work items and approved by designated personnel. How can this be enforced?

A. Manual documentation
B. Environment approval checks, branch policies requiring linked work items, and Azure DevOps audit logs
C. Email approvals
D. Trust the process

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Environment approvals require designated approvers before deployment stages run. Branch policies require linked work items for PR completion. Azure DevOps audit logs provide traceability of all actions. Together they enforce compliance requirements with evidence. Manual documentation (A) and email (C) aren't enforced. Trusting (D) has no verification.

**Key Concept:** [Azure DevOps Audit](https://learn.microsoft.com/en-us/azure/devops/organizations/audit/azure-devops-auditing)
</details>

---

## Domain 5: Implement an Instrumentation Strategy (Questions 35-40)

### Question 35
**Scenario:** A team needs to implement monitoring for their microservices deployed to AKS. They need container metrics, logs, distributed tracing, and Kubernetes events. What should they configure?

A. Basic kubectl logs
B. Azure Monitor Container insights with Application Insights for distributed tracing
C. Third-party monitoring only
D. No monitoring

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Container insights (Azure Monitor) provides container/pod metrics, logs to Log Analytics, and Kubernetes events. Application Insights adds distributed tracing across microservices with correlation. Together they provide comprehensive monitoring. kubectl logs (A) is manual and ephemeral. Third-party (C) may work but adds complexity.

**Key Concept:** [Container Insights](https://learn.microsoft.com/en-us/azure/azure-monitor/containers/container-insights-overview)
</details>

### Question 36
**Scenario:** A team wants to automatically roll back a deployment if error rate exceeds 5% within 10 minutes of deployment. What should they implement?

A. Manual monitoring and rollback
B. Pipeline gates with Azure Monitor query that checks error rate, automatic rollback stage
C. Hope for the best
D. Customer complaints trigger rollback

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Pipeline gates can query Azure Monitor/Application Insights. Post-deployment gate evaluates error rate query. If threshold exceeded, gate fails and subsequent rollback stage can execute. Automated response to production issues. Manual monitoring (A) is slow. Hope (C) and waiting for complaints (D) are not proactive.

**Key Concept:** [Pipeline Gates](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/deploy-using-approvals#add-a-gate)
</details>

### Question 37
**Scenario:** A development team needs to track deployment frequency, lead time, change failure rate, and mean time to recovery (DORA metrics) for their CI/CD process. What should they use?

A. Manual spreadsheet tracking
B. Azure DevOps Analytics with built-in pipeline reports or DORA metrics extension
C. Ignore metrics
D. Count deployments manually

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure DevOps Analytics provides pipeline analytics including pass rate, duration trends, and deployment frequency. DORA metrics extensions or Power BI reports can calculate lead time, change failure rate, and MTTR from pipeline and work item data. Manual tracking (A) is error-prone and time-consuming.

**Key Concept:** [Pipeline Analytics](https://learn.microsoft.com/en-us/azure/devops/pipelines/reports/pipelinereport)
</details>

### Question 38
**Scenario:** A team wants to create custom dashboards showing build success rates, deployment status, work item progress, and Azure resource health in a single view. What should they use?

A. Multiple separate tools
B. Azure DevOps Dashboards with widgets for builds, releases, and Azure Monitor integration
C. Email reports
D. Check each service individually

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure DevOps Dashboards support multiple widget types: build charts, release pipeline status, work item queries, and Azure Monitor widgets for resource metrics. Single pane of glass for DevOps health. Multiple tools (A) and individual checks (D) lack unified view. Email reports (C) aren't real-time.

**Key Concept:** [Azure DevOps Dashboards](https://learn.microsoft.com/en-us/azure/devops/report/dashboards/dashboards)
</details>

### Question 39
**Scenario:** A deployed application is experiencing performance issues. The team needs to identify slow database queries, external API calls, and code-level performance bottlenecks. What should they use?

A. Only review code
B. Application Insights with dependency tracking, profiler, and snapshot debugger
C. User complaints
D. General Azure Monitor metrics

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Application Insights dependency tracking shows slow external calls (database, APIs) with timing. Profiler provides code-level CPU profiling to identify bottlenecks. Snapshot debugger captures exceptions with variable state. Together they enable root cause analysis. Code review (A) doesn't show runtime behavior. Complaints (C) are reactive. General metrics (D) lack code-level detail.

**Key Concept:** [Application Insights Profiler](https://learn.microsoft.com/en-us/azure/azure-monitor/profiler/profiler-overview)
</details>

### Question 40
**Scenario:** A company needs to be alerted when deployments complete (success or failure) and when production exceptions exceed a threshold. Alerts should go to a PagerDuty rotation for on-call engineers. What should they configure?

A. Manual monitoring
B. Azure DevOps service hooks for deployment events, Application Insights alerts, both integrated with PagerDuty
C. Email only
D. Check dashboards hourly

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Service hooks send deployment events to external services including PagerDuty. Application Insights alerts trigger on exception thresholds with PagerDuty action group integration. Both feed into on-call rotation for appropriate response. Manual monitoring (A) and hourly checks (D) miss real-time events. Email (C) may not reach on-call promptly.

**Key Concept:** [Azure Alerting to PagerDuty](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/action-groups)
</details>

---

## Answer Key

| Question | Answer | Domain |
|----------|--------|--------|
| 1 | B | Processes/Communications |
| 2 | B | Processes/Communications |
| 3 | B | Processes/Communications |
| 4 | B | Processes/Communications |
| 5 | B | Processes/Communications |
| 6 | B | Source Control |
| 7 | A | Source Control |
| 8 | B | Source Control |
| 9 | B | Source Control |
| 10 | B | Source Control |
| 11 | B | Source Control |
| 12 | B | Source Control |
| 13 | B | Build/Release Pipelines |
| 14 | B | Build/Release Pipelines |
| 15 | B | Build/Release Pipelines |
| 16 | B | Build/Release Pipelines |
| 17 | B | Build/Release Pipelines |
| 18 | B | Build/Release Pipelines |
| 19 | B | Build/Release Pipelines |
| 20 | B | Build/Release Pipelines |
| 21 | B | Build/Release Pipelines |
| 22 | B | Build/Release Pipelines |
| 23 | B | Build/Release Pipelines |
| 24 | B | Build/Release Pipelines |
| 25 | B | Build/Release Pipelines |
| 26 | B | Build/Release Pipelines |
| 27 | B | Build/Release Pipelines |
| 28 | B | Build/Release Pipelines |
| 29 | C | Build/Release Pipelines |
| 30 | B | Security/Compliance |
| 31 | B | Security/Compliance |
| 32 | B | Security/Compliance |
| 33 | B | Security/Compliance |
| 34 | B | Security/Compliance |
| 35 | B | Instrumentation |
| 36 | B | Instrumentation |
| 37 | B | Instrumentation |
| 38 | B | Instrumentation |
| 39 | B | Instrumentation |
| 40 | B | Instrumentation |
