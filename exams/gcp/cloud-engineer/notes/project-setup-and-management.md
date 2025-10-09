# Project Setup and Management - GCP Associate Cloud Engineer

## Overview

This document covers Google Cloud project setup, organization structure, billing configuration, resource management, and the gcloud CLI. Understanding project and resource management is fundamental for the Associate Cloud Engineer certification.

## Key Topics

### 1. Google Cloud Resource Hierarchy
- Organization, folders, projects, and resources
- Policy inheritance and permissions
- Resource naming and organization
- Best practices for hierarchy design

### 2. Project Management
- Creating and managing projects
- Project settings and configurations
- Quotas and limits
- Project lifecycle management

### 3. Billing Management
- Billing accounts and budgets
- Cost management and optimization
- Billing reports and export
- Budget alerts and notifications

### 4. gcloud CLI
- Installation and configuration
- Authentication and authorization
- Common commands and patterns
- Configuration management

### 5. APIs and Services
- Enabling and disabling APIs
- API quotas and rate limits
- Service accounts for API access
- API authentication methods

## GCP Services Reference

### Resource Hierarchy
- **Organization**: Top-level container for all resources
- **Folders**: Grouping mechanism for projects
- **Projects**: Base container for resources and billing
- **Resources**: Individual services and assets (VMs, buckets, etc.)
- **Labels**: Key-value pairs for resource organization
- **Tags**: Network-level resource identification

### Project Management
- **Project ID**: Globally unique, permanent identifier
- **Project Name**: Human-readable, changeable display name
- **Project Number**: Auto-generated numeric identifier
- **IAM Policies**: Access control at project level
- **Quotas**: Limits on resource usage
- **APIs & Services**: Enabled APIs for the project

### Billing
- **Billing Account**: Pays for project resource usage
- **Budgets**: Spending limits and alerts
- **Reports**: Cost analysis and breakdowns
- **Exports**: Send billing data to BigQuery
- **SKUs**: Individual billable items
- **Committed Use Discounts**: Long-term usage commitments
- **Sustained Use Discounts**: Automatic discounts for consistent usage

### gcloud CLI
- **Configurations**: Named sets of settings
- **Properties**: Settings like project, region, zone
- **Authentication**: User and service account credentials
- **Components**: Additional tools and extensions
- **Output Formats**: JSON, YAML, table, CSV

## Best Practices

### Organization Hierarchy Best Practices
1. **Use Organizations**: Create organization for centralized management
2. **Folder Structure**: Organize by department, environment, or team
3. **Environment Separation**: Use separate folders for dev, staging, production
4. **Naming Conventions**: Implement consistent naming across resources
5. **Policy Inheritance**: Apply common policies at organization/folder level
6. **Service Projects**: Use Shared VPC and service projects for networking
7. **Resource Labels**: Apply labels for cost allocation and organization
8. **Avoid Flat Structure**: Use hierarchy to simplify management

### Project Management Best Practices
1. **Project per Environment**: Separate projects for dev, staging, production
2. **Project Naming**: Use descriptive, consistent naming conventions
3. **Project Numbers**: Use for programmatic access (more stable than IDs)
4. **Quota Management**: Monitor and request increases proactively
5. **API Management**: Only enable required APIs
6. **Lifecycle Management**: Archive or delete unused projects
7. **Access Control**: Apply least privilege principle
8. **Documentation**: Document project purpose and ownership

### Billing Best Practices
1. **Budget Alerts**: Set up alerts at multiple thresholds (50%, 90%, 100%)
2. **Cost Allocation**: Use labels for department/team cost tracking
3. **Regular Review**: Review billing reports weekly or monthly
4. **Commitment Discounts**: Purchase for predictable workloads
5. **Preemptible VMs**: Use for fault-tolerant workloads (up to 80% savings)
6. **Idle Resource Detection**: Monitor and eliminate unused resources
7. **Billing Exports**: Export to BigQuery for detailed analysis
8. **Multiple Billing Accounts**: Separate by department or business unit

### gcloud CLI Best Practices
1. **Use Configurations**: Create separate configurations for different contexts
2. **Default Settings**: Set default project, region, and zone
3. **Service Accounts**: Use for automation and CI/CD
4. **Command Aliases**: Create aliases for frequently used commands
5. **Output Formats**: Use JSON for scripting, table for human readability
6. **Interactive Mode**: Use --interactive flag for confirmation prompts
7. **Filter and Format**: Use --filter and --format for precise output
8. **Keep Updated**: Regularly update gcloud components

### Security and Compliance Best Practices
1. **Audit Logging**: Enable all audit log types
2. **Organization Policies**: Enforce constraints at org level
3. **Service Account Keys**: Minimize use, prefer Workload Identity
4. **Regular Audits**: Review IAM policies and access regularly
5. **Resource Constraints**: Use organization policies to prevent risky configurations
6. **Separation of Duties**: Separate roles for security, billing, and operations
7. **MFA Enforcement**: Require multi-factor authentication
8. **Compliance Monitoring**: Use Security Command Center

## Common Scenarios

### Scenario 1: Multi-Environment Setup
**Requirement**: Set up dev, staging, and production environments
**Solution**:
- Create organization with folders for each environment
- Create separate projects within each folder
- Configure separate billing for production
- Apply environment-specific IAM policies at folder level
- Use consistent naming convention (project-env)

### Scenario 2: Cost Control and Monitoring
**Requirement**: Control costs and receive spending alerts
**Solution**:
- Create billing budgets with multiple alert thresholds
- Set up billing exports to BigQuery
- Apply resource labels for cost allocation
- Create custom reports for cost analysis
- Implement automated resource cleanup for dev/test

### Scenario 3: Centralized Network Management
**Requirement**: Manage networking across multiple projects
**Solution**:
- Create host project with Shared VPC
- Attach service projects to Shared VPC
- Centrally manage firewall rules and subnets
- Use IAM for granular network administration
- Implement VPC Flow Logs for monitoring

### Scenario 4: Developer Access Management
**Requirement**: Provide developers with appropriate access
**Solution**:
- Create Google Groups for different teams
- Assign predefined roles to groups (not individual users)
- Use custom roles for specific needs
- Grant access at project or folder level
- Implement time-bound access for elevated privileges

### Scenario 5: CI/CD Pipeline Setup
**Requirement**: Automate deployments using gcloud CLI
**Solution**:
- Create service account for CI/CD
- Grant minimum necessary permissions
- Use service account for gcloud authentication
- Create gcloud configuration for automation
- Store credentials securely in secret management system

## Study Tips

### Hands-On Practice
1. **Create Organization**: Set up organization hierarchy with folders
2. **Manage Projects**: Create, configure, and manage multiple projects
3. **Configure Billing**: Set up billing accounts and budgets
4. **Use gcloud CLI**: Practice common commands and configurations
5. **Apply Labels**: Add and manage labels on resources

### Key Concepts to Master
1. **Resource Hierarchy**: Organization → Folders → Projects → Resources
2. **Policy Inheritance**: How IAM policies flow down the hierarchy
3. **Billing Structure**: Billing accounts, projects, and cost allocation
4. **gcloud Syntax**: Understanding command structure and flags
5. **Project Quotas**: Types of quotas and how to manage them

### Common Exam Topics
1. Creating and managing projects and folders
2. Setting up and managing billing accounts
3. Configuring budgets and alerts
4. Using gcloud CLI for resource management
5. Enabling and managing APIs and services
6. Understanding resource hierarchy and policy inheritance
7. Applying labels for resource organization
8. Managing quotas and requesting increases

### gcloud Command Examples

```bash
# Authentication and Configuration
gcloud auth login
gcloud auth list
gcloud auth application-default login
gcloud config set project PROJECT_ID
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-a
gcloud config list

# Configuration Management
gcloud config configurations create dev-config
gcloud config configurations activate dev-config
gcloud config configurations list
gcloud config configurations describe dev-config

# Project Management
gcloud projects create PROJECT_ID --name="My Project"
gcloud projects list
gcloud projects describe PROJECT_ID
gcloud projects delete PROJECT_ID
gcloud projects add-iam-policy-binding PROJECT_ID --member=user:email@example.com --role=roles/editor

# Get current project
gcloud config get-value project

# Billing
gcloud billing accounts list
gcloud billing projects link PROJECT_ID --billing-account=ACCOUNT_ID
gcloud billing projects describe PROJECT_ID

# Organization and Folders
gcloud organizations list
gcloud resource-manager folders create --display-name="Development" --organization=ORG_ID
gcloud resource-manager folders list --organization=ORG_ID

# API Management
gcloud services enable compute.googleapis.com
gcloud services list --enabled
gcloud services list --available
gcloud services disable SERVICE_NAME

# Quotas
gcloud compute project-info describe --project=PROJECT_ID
gcloud compute regions describe us-central1

# Labels
gcloud compute instances add-labels INSTANCE_NAME --labels=env=prod,team=backend --zone=us-central1-a
gcloud compute instances update INSTANCE_NAME --update-labels=version=v2 --zone=us-central1-a
gcloud compute instances remove-labels INSTANCE_NAME --labels=env --zone=us-central1-a

# List resources with labels
gcloud compute instances list --filter="labels.env=prod"

# Service Accounts for Automation
gcloud iam service-accounts create ci-cd-sa --display-name="CI/CD Service Account"
gcloud iam service-accounts keys create key.json --iam-account=ci-cd-sa@PROJECT_ID.iam.gserviceaccount.com
gcloud auth activate-service-account --key-file=key.json

# General Management
gcloud info
gcloud version
gcloud components list
gcloud components update
gcloud components install kubectl

# Help and Documentation
gcloud help
gcloud compute instances create --help
gcloud help compute instances

# Output Formatting
gcloud compute instances list --format=json
gcloud compute instances list --format=yaml
gcloud compute instances list --format="table(name,zone,status)"
gcloud compute instances list --format="value(name,zone)"

# Filtering
gcloud compute instances list --filter="zone:us-central1-a"
gcloud compute instances list --filter="status=RUNNING"
gcloud compute instances list --filter="labels.env=prod AND zone:us-central1"

# Interactive Mode
gcloud compute instances create --interactive
```

### Environment Variables

```bash
# Set default project
export CLOUDSDK_CORE_PROJECT=my-project

# Set default region/zone
export CLOUDSDK_COMPUTE_REGION=us-central1
export CLOUDSDK_COMPUTE_ZONE=us-central1-a

# Disable prompts for scripting
export CLOUDSDK_CORE_DISABLE_PROMPTS=1

# Set output format
export CLOUDSDK_CORE_FORMAT=json
```

## Additional Resources

- [Resource Manager Documentation](https://cloud.google.com/resource-manager/docs)
- [Billing Documentation](https://cloud.google.com/billing/docs)
- [gcloud CLI Documentation](https://cloud.google.com/sdk/gcloud)
- [Organization Policies](https://cloud.google.com/resource-manager/docs/organization-policy/overview)
- [Resource Hierarchy Best Practices](https://cloud.google.com/docs/enterprise/best-practices-for-enterprise-organizations)
- [Cost Optimization](https://cloud.google.com/architecture/best-practices-for-cost-optimization)
