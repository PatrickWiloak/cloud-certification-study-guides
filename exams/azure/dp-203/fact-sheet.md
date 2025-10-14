# Azure DP-203: Data Engineering on Microsoft Azure - Fact Sheet

## Exam Overview

**Exam DP-203: Data Engineering on Microsoft Azure** validates skills in designing and implementing data solutions using Azure services. This certification demonstrates expertise in integrating, transforming, and consolidating data from various sources into analytics solutions.

### Exam Details
- **Duration**: 120 minutes
- **Question Types**: Multiple choice, multiple select, case studies, drag-and-drop
- **Passing Score**: 700/1000
- **Cost**: $165 USD
- **Languages**: English, Japanese, Chinese (Simplified), Korean, German, French, Spanish, Portuguese (Brazil), Russian, Indonesian, Arabic, Chinese (Traditional), Italian

### Exam Domains (Skills Measured)
1. **Design and implement data storage** (15-20%)
2. **Develop data processing** (40-45%)
3. **Secure, monitor, and optimize data storage and data processing** (30-35%)

---

## 1. Azure Data Storage Solutions

### Azure Data Lake Storage Gen2

**[📖 Azure Data Lake Storage Gen2 Introduction](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-introduction)** - Overview of hierarchical namespace and big data analytics capabilities

**[📖 Data Lake Storage Gen2 Best Practices](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-best-practices)** - Performance optimization and design patterns

**[📖 Hierarchical Namespace](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-namespace)** - Understanding file and directory organization

**[📖 Access Control Lists (ACLs)](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-access-control)** - POSIX-style permissions for files and directories

**[📖 Multi-Protocol Access](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-multi-protocol-access)** - Blob API and ADLS Gen2 API compatibility

**[📖 Performance Tuning Guide](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-performance-tuning-guidance)** - Optimizing throughput and latency

**[📖 Lifecycle Management Policies](https://docs.microsoft.com/azure/storage/blobs/lifecycle-management-overview)** - Automating data tiering and deletion

**[📖 Access Tiers](https://docs.microsoft.com/azure/storage/blobs/access-tiers-overview)** - Hot, Cool, Cold, and Archive storage tiers

**[📖 Security Recommendations](https://docs.microsoft.com/azure/storage/blobs/security-recommendations)** - Securing data lake storage accounts

**[📖 Disaster Recovery](https://docs.microsoft.com/azure/storage/common/storage-disaster-recovery-guidance)** - High availability and failover strategies

### Azure Synapse Analytics

**[📖 Azure Synapse Analytics Overview](https://docs.microsoft.com/azure/synapse-analytics/overview-what-is)** - Unified analytics platform for data warehousing and big data

**[📖 Dedicated SQL Pools](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is)** - Massively parallel processing (MPP) architecture

**[📖 Serverless SQL Pools](https://docs.microsoft.com/azure/synapse-analytics/sql/on-demand-workspace-overview)** - Query data in data lake without provisioning infrastructure

**[📖 Apache Spark Pools](https://docs.microsoft.com/azure/synapse-analytics/spark/apache-spark-overview)** - In-memory big data processing with Spark

**[📖 Data Integration Pipelines](https://docs.microsoft.com/azure/synapse-analytics/get-started-pipelines)** - ETL/ELT orchestration within Synapse

**[📖 Synapse Studio](https://docs.microsoft.com/azure/synapse-analytics/get-started)** - Unified workspace for data engineering tasks

**[📖 Table Distribution Strategies](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-distribute)** - Round-robin, hash, and replicated distribution

**[📖 Table Indexing](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-index)** - Clustered columnstore, clustered, and nonclustered indexes

**[📖 Partitioning Tables](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-partition)** - Improving query performance with partitioning

**[📖 Resource Classes and Workload Management](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/resource-classes-for-workload-management)** - Managing query concurrency and resources

**[📖 PolyBase](https://docs.microsoft.com/azure/synapse-analytics/sql/load-data-overview)** - Loading and querying external data sources

**[📖 COPY Statement](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/quickstart-bulk-load-copy-tsql)** - High-throughput data ingestion

**[📖 Delta Lake Support](https://docs.microsoft.com/azure/synapse-analytics/spark/apache-spark-delta-lake-overview)** - ACID transactions for data lakes

**[📖 Link Feature](https://docs.microsoft.com/azure/synapse-analytics/synapse-link/concept-synapse-link)** - Near real-time analytics over operational data

### Azure Cosmos DB

**[📖 Azure Cosmos DB Overview](https://docs.microsoft.com/azure/cosmos-db/introduction)** - Globally distributed, multi-model database service

**[📖 Consistency Levels](https://docs.microsoft.com/azure/cosmos-db/consistency-levels)** - Strong, bounded staleness, session, consistent prefix, eventual

**[📖 Partitioning Strategy](https://docs.microsoft.com/azure/cosmos-db/partitioning-overview)** - Logical and physical partitions

**[📖 Request Units (RUs)](https://docs.microsoft.com/azure/cosmos-db/request-units)** - Understanding throughput pricing model

**[📖 Change Feed](https://docs.microsoft.com/azure/cosmos-db/change-feed)** - Processing data changes in real-time

**[📖 Analytical Store](https://docs.microsoft.com/azure/cosmos-db/analytical-store-introduction)** - Column-oriented storage for analytics

**[📖 Synapse Link for Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/synapse-link)** - Hybrid transactional and analytical processing (HTAP)

**[📖 Global Distribution](https://docs.microsoft.com/azure/cosmos-db/distribute-data-globally)** - Multi-region writes and reads

**[📖 Indexing Policies](https://docs.microsoft.com/azure/cosmos-db/index-policy)** - Automatic and custom indexing strategies

**[📖 Time to Live (TTL)](https://docs.microsoft.com/azure/cosmos-db/time-to-live)** - Automatic data expiration

### Azure SQL Database

**[📖 Azure SQL Database Overview](https://docs.microsoft.com/azure/azure-sql/database/sql-database-paas-overview)** - Intelligent, scalable cloud database service

**[📖 Service Tiers](https://docs.microsoft.com/azure/azure-sql/database/service-tiers-general-purpose-business-critical)** - DTU and vCore purchasing models

**[📖 Elastic Pools](https://docs.microsoft.com/azure/azure-sql/database/elastic-pool-overview)** - Resource sharing across multiple databases

**[📖 Hyperscale Service Tier](https://docs.microsoft.com/azure/azure-sql/database/service-tier-hyperscale)** - Highly scalable storage and compute

**[📖 Geo-Replication](https://docs.microsoft.com/azure/azure-sql/database/active-geo-replication-overview)** - Active geo-replication and failover groups

---

## 2. Data Processing and Transformation

### Azure Data Factory

**[📖 Azure Data Factory Overview](https://docs.microsoft.com/azure/data-factory/introduction)** - Cloud-based ETL and data integration service

**[📖 Pipelines and Activities](https://docs.microsoft.com/azure/data-factory/concepts-pipelines-activities)** - Orchestrating data movement and transformation

**[📖 Linked Services](https://docs.microsoft.com/azure/data-factory/concepts-linked-services)** - Connection information to data stores

**[📖 Datasets](https://docs.microsoft.com/azure/data-factory/concepts-datasets-linked-services)** - Data structure references within stores

**[📖 Integration Runtime](https://docs.microsoft.com/azure/data-factory/concepts-integration-runtime)** - Compute infrastructure for data integration

**[📖 Mapping Data Flows](https://docs.microsoft.com/azure/data-factory/concepts-data-flow-overview)** - Visual data transformation at scale

**[📖 Wrangling Data Flows](https://docs.microsoft.com/azure/data-factory/wrangling-overview)** - Code-free data preparation with Power Query

**[📖 Control Flow Activities](https://docs.microsoft.com/azure/data-factory/concepts-pipelines-activities#control-flow-activities)** - Conditional execution, loops, and branching

**[📖 Triggers](https://docs.microsoft.com/azure/data-factory/concepts-pipeline-execution-triggers)** - Schedule, tumbling window, and event-based triggers

**[📖 Parameters and Expressions](https://docs.microsoft.com/azure/data-factory/control-flow-expression-language-functions)** - Dynamic pipeline configuration

**[📖 Copy Activity](https://docs.microsoft.com/azure/data-factory/copy-activity-overview)** - Data movement between stores

**[📖 Lookup Activity](https://docs.microsoft.com/azure/data-factory/control-flow-lookup-activity)** - Retrieving configuration data

**[📖 ForEach Activity](https://docs.microsoft.com/azure/data-factory/control-flow-for-each-activity)** - Iterating over collections

**[📖 Execute Pipeline Activity](https://docs.microsoft.com/azure/data-factory/control-flow-execute-pipeline-activity)** - Modular pipeline design

**[📖 Schema Drift Handling](https://docs.microsoft.com/azure/data-factory/concepts-data-flow-schema-drift)** - Managing evolving data structures

**[📖 Performance Optimization](https://docs.microsoft.com/azure/data-factory/copy-activity-performance)** - Tuning data movement performance

**[📖 Fault Tolerance](https://docs.microsoft.com/azure/data-factory/copy-activity-fault-tolerance)** - Handling incompatible rows

**[📖 Incremental Loading](https://docs.microsoft.com/azure/data-factory/tutorial-incremental-copy-overview)** - Loading only changed data

### Azure Databricks

**[📖 Azure Databricks Overview](https://docs.microsoft.com/azure/databricks/scenarios/what-is-azure-databricks)** - Apache Spark-based analytics platform

**[📖 Workspace Organization](https://docs.microsoft.com/azure/databricks/workspace/)** - Notebooks, clusters, jobs, and libraries

**[📖 Clusters](https://docs.microsoft.com/azure/databricks/clusters/)** - Interactive and job clusters configuration

**[📖 Autoscaling](https://docs.microsoft.com/azure/databricks/clusters/configure#autoscaling)** - Dynamic cluster scaling

**[📖 Notebooks](https://docs.microsoft.com/azure/databricks/notebooks/)** - Interactive development with Python, Scala, R, SQL

**[📖 Jobs](https://docs.microsoft.com/azure/databricks/jobs)** - Scheduling and orchestrating workflows

**[📖 Delta Lake](https://docs.microsoft.com/azure/databricks/delta/)** - ACID transactions for data lakes

**[📖 Delta Table Optimization](https://docs.microsoft.com/azure/databricks/delta/optimize)** - Compaction and Z-ordering

**[📖 Time Travel](https://docs.microsoft.com/azure/databricks/delta/versioning)** - Querying historical versions of data

**[📖 Structured Streaming](https://docs.microsoft.com/azure/databricks/structured-streaming/)** - Real-time data processing with Spark

**[📖 Auto Loader](https://docs.microsoft.com/azure/databricks/ingestion/auto-loader/)** - Incremental file processing

**[📖 Unity Catalog](https://docs.microsoft.com/azure/databricks/data-governance/unity-catalog/)** - Unified governance for data and AI

**[📖 Secret Management](https://docs.microsoft.com/azure/databricks/security/secrets/)** - Securely storing credentials

**[📖 MLflow Integration](https://docs.microsoft.com/azure/databricks/mlflow/)** - Machine learning lifecycle management

### Azure Stream Analytics

**[📖 Stream Analytics Overview](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-introduction)** - Real-time analytics on streaming data

**[📖 Inputs](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-add-inputs)** - Event Hubs, IoT Hub, Blob storage

**[📖 Outputs](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-define-outputs)** - SQL Database, Blob storage, Event Hubs, Power BI

**[📖 Query Language](https://docs.microsoft.com/stream-analytics-query/stream-analytics-query-language-reference)** - SQL-like syntax for stream processing

**[📖 Windowing Functions](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-window-functions)** - Tumbling, hopping, sliding, and session windows

**[📖 Streaming Units (SUs)](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-streaming-unit-consumption)** - Compute capacity allocation

**[📖 Time Policies](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-time-handling)** - Event time vs. arrival time

**[📖 Late Arrival and Out-of-Order Events](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-out-of-order-and-late-events)** - Handling event timing issues

**[📖 User-Defined Functions](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-javascript-user-defined-functions)** - JavaScript UDFs for custom logic

**[📖 Geospatial Functions](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-geospatial-functions)** - Location-based analytics

**[📖 Compatibility Level](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-compatibility-level)** - Feature availability across versions

### Azure Event Hubs

**[📖 Event Hubs Overview](https://docs.microsoft.com/azure/event-hubs/event-hubs-about)** - Big data streaming platform and event ingestion service

**[📖 Partitions](https://docs.microsoft.com/azure/event-hubs/event-hubs-features#partitions)** - Parallel processing and ordering guarantees

**[📖 Consumer Groups](https://docs.microsoft.com/azure/event-hubs/event-hubs-features#consumer-groups)** - Multiple readers on the same stream

**[📖 Capture Feature](https://docs.microsoft.com/azure/event-hubs/event-hubs-capture-overview)** - Automatic data archival to storage

**[📖 Throughput Units](https://docs.microsoft.com/azure/event-hubs/event-hubs-scalability#throughput-units)** - Capacity planning for standard tier

**[📖 Auto-Inflate](https://docs.microsoft.com/azure/event-hubs/event-hubs-auto-inflate)** - Automatic scaling of throughput units

**[📖 Event Hubs Dedicated](https://docs.microsoft.com/azure/event-hubs/event-hubs-dedicated-overview)** - Single-tenant deployments

**[📖 Apache Kafka Integration](https://docs.microsoft.com/azure/event-hubs/event-hubs-for-kafka-ecosystem-overview)** - Kafka protocol support

**[📖 Schema Registry](https://docs.microsoft.com/azure/event-hubs/schema-registry-overview)** - Schema validation and evolution

### Azure Functions

**[📖 Azure Functions Overview](https://docs.microsoft.com/azure/azure-functions/functions-overview)** - Serverless compute for event-driven applications

**[📖 Triggers and Bindings](https://docs.microsoft.com/azure/azure-functions/functions-triggers-bindings)** - Declarative connections to services

**[📖 Durable Functions](https://docs.microsoft.com/azure/azure-functions/durable/durable-functions-overview)** - Stateful workflows in serverless

**[📖 Event Hub Trigger](https://docs.microsoft.com/azure/azure-functions/functions-bindings-event-hubs-trigger)** - Processing streaming events

**[📖 Cosmos DB Trigger](https://docs.microsoft.com/azure/azure-functions/functions-bindings-cosmosdb-v2-trigger)** - Change feed processing

**[📖 Timer Trigger](https://docs.microsoft.com/azure/azure-functions/functions-bindings-timer)** - Schedule-based execution

---

## 3. Data Security and Governance

### Authentication and Authorization

**[📖 Azure Active Directory Integration](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-whatis)** - Identity and access management

**[📖 Managed Identities](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview)** - System-assigned and user-assigned identities

**[📖 Service Principal](https://docs.microsoft.com/azure/active-directory/develop/app-objects-and-service-principals)** - Application identity for resource access

**[📖 Azure RBAC](https://docs.microsoft.com/azure/role-based-access-control/overview)** - Role-based access control for Azure resources

**[📖 Storage Account Keys](https://docs.microsoft.com/azure/storage/common/storage-account-keys-manage)** - Managing shared access keys

**[📖 Shared Access Signatures (SAS)](https://docs.microsoft.com/azure/storage/common/storage-sas-overview)** - Delegated access to storage resources

**[📖 Account SAS vs Service SAS](https://docs.microsoft.com/azure/storage/common/storage-sas-overview#types-of-shared-access-signatures)** - Different SAS token types

**[📖 Stored Access Policy](https://docs.microsoft.com/azure/storage/common/storage-stored-access-policy-define-dotnet)** - Centralized SAS management

### Encryption and Key Management

**[📖 Encryption at Rest](https://docs.microsoft.com/azure/security/fundamentals/encryption-atrest)** - Data protection when stored

**[📖 Encryption in Transit](https://docs.microsoft.com/azure/security/fundamentals/encryption-overview#encryption-of-data-in-transit)** - TLS/SSL for data movement

**[📖 Azure Key Vault](https://docs.microsoft.com/azure/key-vault/general/overview)** - Secrets, keys, and certificates management

**[📖 Customer-Managed Keys](https://docs.microsoft.com/azure/storage/common/customer-managed-keys-overview)** - Bring your own encryption keys

**[📖 Transparent Data Encryption](https://docs.microsoft.com/azure/azure-sql/database/transparent-data-encryption-tde-overview)** - SQL Database encryption

**[📖 Always Encrypted](https://docs.microsoft.com/azure/azure-sql/database/always-encrypted-azure-key-vault-configure)** - Column-level encryption in SQL

### Network Security

**[📖 Virtual Networks](https://docs.microsoft.com/azure/virtual-network/virtual-networks-overview)** - Network isolation for Azure resources

**[📖 Service Endpoints](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview)** - Direct routing to Azure services

**[📖 Private Endpoints](https://docs.microsoft.com/azure/private-link/private-endpoint-overview)** - Private IP access to Azure services

**[📖 Firewall Rules](https://docs.microsoft.com/azure/storage/common/storage-network-security)** - IP-based access restrictions

**[📖 Azure Private Link](https://docs.microsoft.com/azure/private-link/private-link-overview)** - Accessing services over private connection

### Data Governance and Compliance

**[📖 Azure Purview Overview](https://docs.microsoft.com/azure/purview/overview)** - Unified data governance service

**[📖 Data Catalog](https://docs.microsoft.com/azure/purview/overview#data-catalog)** - Discovering and understanding data assets

**[📖 Data Lineage](https://docs.microsoft.com/azure/purview/concept-data-lineage)** - Tracking data origin and transformations

**[📖 Data Classification](https://docs.microsoft.com/azure/purview/concept-best-practices-classification)** - Automated and manual sensitivity labeling

**[📖 Dynamic Data Masking](https://docs.microsoft.com/azure/azure-sql/database/dynamic-data-masking-overview)** - Limiting sensitive data exposure

**[📖 Row-Level Security](https://docs.microsoft.com/sql/relational-databases/security/row-level-security)** - Restricting row access in tables

**[📖 Column-Level Security](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/column-level-security)** - Controlling column access

---

## 4. Monitoring and Optimization

### Azure Monitor

**[📖 Azure Monitor Overview](https://docs.microsoft.com/azure/azure-monitor/overview)** - Full-stack monitoring for Azure resources

**[📖 Metrics](https://docs.microsoft.com/azure/azure-monitor/essentials/data-platform-metrics)** - Time-series performance data

**[📖 Logs](https://docs.microsoft.com/azure/azure-monitor/logs/data-platform-logs)** - Detailed diagnostic and operational data

**[📖 Log Analytics Workspace](https://docs.microsoft.com/azure/azure-monitor/logs/log-analytics-workspace-overview)** - Centralized log storage and querying

**[📖 KQL (Kusto Query Language)](https://docs.microsoft.com/azure/data-explorer/kusto/query/)** - Query language for log analytics

**[📖 Alerts](https://docs.microsoft.com/azure/azure-monitor/alerts/alerts-overview)** - Proactive notification on conditions

**[📖 Action Groups](https://docs.microsoft.com/azure/azure-monitor/alerts/action-groups)** - Notifications and automated responses

**[📖 Diagnostic Settings](https://docs.microsoft.com/azure/azure-monitor/essentials/diagnostic-settings)** - Routing platform logs and metrics

**[📖 Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview)** - Application performance monitoring

### Performance Optimization

**[📖 Query Performance Insight](https://docs.microsoft.com/azure/azure-sql/database/query-performance-insight-use)** - SQL Database query analysis

**[📖 Automatic Tuning](https://docs.microsoft.com/azure/azure-sql/database/automatic-tuning-overview)** - AI-powered database optimization

**[📖 Index Advisor](https://docs.microsoft.com/azure/azure-sql/database/database-advisor-implement-performance-recommendations)** - Index recommendations

**[📖 Materialized Views](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/performance-tuning-materialized-views)** - Pre-computed aggregations

**[📖 Result Set Caching](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/performance-tuning-result-set-caching)** - Caching query results

**[📖 Partition Elimination](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-partition)** - Reducing data scanned

**[📖 Statistics](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-statistics)** - Query optimizer input data

**[📖 Dynamic Management Views](https://docs.microsoft.com/sql/relational-databases/system-dynamic-management-views/system-dynamic-management-views)** - System health and performance

### Cost Management

**[📖 Azure Cost Management](https://docs.microsoft.com/azure/cost-management-billing/costs/overview-cost-management)** - Analyzing and optimizing spending

**[📖 Budgets and Alerts](https://docs.microsoft.com/azure/cost-management-billing/costs/tutorial-acm-create-budgets)** - Spending limits and notifications

**[📖 Pricing Calculator](https://azure.microsoft.com/pricing/calculator/)** - Estimating Azure service costs

**[📖 Reserved Capacity](https://docs.microsoft.com/azure/cost-management-billing/reservations/save-compute-costs-reservations)** - Discounted pricing with commitments

---

## 5. Data Formats and Serialization

### File Formats

**[📖 Parquet Format](https://parquet.apache.org/documentation/latest/)** - Columnar storage format for analytics

**[📖 Avro Format](https://avro.apache.org/docs/)** - Row-based serialization with schema evolution

**[📖 ORC Format](https://orc.apache.org/)** - Optimized row columnar format

**[📖 JSON Format](https://www.json.org/)** - Human-readable text-based format

**[📖 CSV Format Best Practices](https://docs.microsoft.com/azure/data-factory/format-delimited-text)** - Delimited text file handling

### Compression

**[📖 Compression in Data Lake](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-best-practices#compression)** - Choosing compression codecs

**[📖 Snappy Compression](https://google.github.io/snappy/)** - Fast compression for Hadoop workloads

**[📖 Gzip Compression](https://www.gnu.org/software/gzip/)** - High compression ratio format

---

## 6. Best Practices and Design Patterns

### Data Ingestion Patterns

**[📖 Batch Processing](https://docs.microsoft.com/azure/architecture/data-guide/big-data/#batch-processing)** - Processing data in scheduled intervals

**[📖 Stream Processing](https://docs.microsoft.com/azure/architecture/data-guide/big-data/#real-time-processing)** - Continuous data processing

**[📖 Lambda Architecture](https://docs.microsoft.com/azure/architecture/data-guide/big-data/non-relational-data#lambda-architecture)** - Batch and speed layers combined

**[📖 Kappa Architecture](https://docs.microsoft.com/azure/architecture/data-guide/big-data/real-time-processing#kappa-architecture)** - Stream processing only approach

**[📖 Medallion Architecture](https://docs.microsoft.com/azure/databricks/lakehouse/medallion)** - Bronze, silver, gold data layers

### Data Modeling

**[📖 Star Schema](https://docs.microsoft.com/power-bi/guidance/star-schema)** - Dimensional modeling with fact and dimension tables

**[📖 Snowflake Schema](https://docs.microsoft.com/analysis-services/multidimensional-models/multidimensional-model-databases-ssas#snowflake-schema)** - Normalized dimension tables

**[📖 Slowly Changing Dimensions](https://docs.microsoft.com/azure/data-factory/tutorial-incremental-copy-change-data-capture-feature-portal#slowly-changing-dimensions)** - Handling dimension updates

**[📖 Data Vault](https://docs.microsoft.com/sql/relational-databases/tables/temporal-tables)** - Agile data warehouse modeling

### High Availability and Disaster Recovery

**[📖 High Availability Design](https://docs.microsoft.com/azure/architecture/framework/resiliency/overview)** - Architecting resilient solutions

**[📖 Backup and Restore](https://docs.microsoft.com/azure/azure-sql/database/automated-backups-overview)** - Automated backup strategies

**[📖 Business Continuity](https://docs.microsoft.com/azure/architecture/framework/resiliency/backup-and-recovery)** - Ensuring service continuity

**[📖 Geo-Redundancy](https://docs.microsoft.com/azure/storage/common/storage-redundancy)** - LRS, ZRS, GRS, RA-GRS, GZRS options

---

## 7. Exam Preparation Tips

### Key Study Areas

1. **Understand service capabilities and limitations**: Know when to use each Azure service
2. **Hands-on practice**: Create resources and implement solutions in Azure portal
3. **Performance optimization**: Learn distribution, indexing, and partitioning strategies
4. **Security implementation**: Practice configuring authentication, encryption, and network security
5. **Monitoring and troubleshooting**: Use Azure Monitor, diagnostic logs, and KQL queries
6. **Cost optimization**: Understand pricing models and cost-saving features
7. **Design patterns**: Learn common architectures for batch and stream processing

### Practice Resources

**[📖 Microsoft Learn DP-203 Path](https://docs.microsoft.com/learn/certifications/exams/dp-203)** - Official Microsoft learning path

**[📖 Azure Free Account](https://azure.microsoft.com/free/)** - $200 credit for 30 days

**[📖 Azure Sandbox Environment](https://docs.microsoft.com/learn/support/faq#what-are-microsoft-learn-sandboxes-)** - Practice without Azure subscription

**[📖 Exam Skills Outline](https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RE4MbYT)** - Detailed exam objectives PDF

### Common Exam Scenarios

1. **Data ingestion from multiple sources**: Choosing appropriate tools and patterns
2. **Transforming and cleansing data**: Using Data Factory, Databricks, or Synapse
3. **Designing storage solutions**: Selecting storage types and configurations
4. **Implementing security**: Authentication, authorization, and encryption
5. **Optimizing query performance**: Indexing, partitioning, and caching
6. **Monitoring data pipelines**: Setting up alerts and diagnostics
7. **Handling streaming data**: Event Hubs and Stream Analytics configuration
8. **Implementing disaster recovery**: Backup, geo-replication, and failover

---

## 8. Important Concepts Summary

### Data Storage Concepts
- **Hierarchical namespace**: File system semantics for data lakes
- **Partition keys**: Distribution strategy for parallel processing
- **Consistency levels**: Trade-offs between performance and data consistency
- **Data redundancy**: LRS, ZRS, GRS, RA-GRS, GZRS options
- **Access tiers**: Hot, Cool, Cold, Archive for cost optimization
- **ACLs vs RBAC**: File-level vs resource-level permissions

### Data Processing Concepts
- **MPP architecture**: Massively parallel processing in Synapse
- **Serverless computing**: On-demand compute without infrastructure management
- **Pipeline orchestration**: Dependencies, triggers, and control flow
- **Schema drift**: Handling evolving data structures
- **Windowing**: Tumbling, hopping, sliding, session windows
- **Event time vs arrival time**: Temporal processing considerations

### Security Concepts
- **Defense in depth**: Multiple layers of security
- **Least privilege**: Minimum necessary permissions
- **Zero trust**: Verify explicitly, assume breach
- **Data encryption**: At rest and in transit
- **Network isolation**: VNets, private endpoints, service endpoints
- **Identity management**: AAD, managed identities, service principals

### Monitoring Concepts
- **Telemetry collection**: Metrics, logs, traces
- **KQL queries**: Analyzing log data
- **Alert rules**: Metric, log, activity log alerts
- **Diagnostic settings**: Routing platform logs
- **Performance baselines**: Establishing normal behavior
- **Cost analysis**: Understanding spending patterns

---

## 9. Quick Reference Commands

### Azure CLI Commands
```bash
# Data Factory
az datafactory create
az datafactory pipeline create-run
az datafactory pipeline-run show

# Synapse Analytics
az synapse workspace create
az synapse spark pool create
az synapse sql pool create

# Storage Account
az storage account create
az storage blob upload-batch
az storage account keys list

# Databricks
az databricks workspace create
az databricks workspace update

# Event Hubs
az eventhubs namespace create
az eventhubs eventhub create
az eventhubs eventhub consumer-group create

# Stream Analytics
az stream-analytics job create
az stream-analytics input create
az stream-analytics output create
```

### PowerShell Commands
```powershell
# Resource Management
New-AzResourceGroup
New-AzStorageAccount
New-AzDataFactoryV2

# Synapse
New-AzSynapseWorkspace
New-AzSynapseSqlPool
New-AzSynapseSparkPool

# Monitor
New-AzMetricAlertRuleV2
Get-AzLog
New-AzActionGroup

# Key Vault
New-AzKeyVault
Set-AzKeyVaultSecret
Get-AzKeyVaultSecret
```

### SQL Queries for Synapse
```sql
-- Create external data source
CREATE EXTERNAL DATA SOURCE DataLakeSource
WITH (LOCATION = 'abfss://container@account.dfs.core.windows.net');

-- Create external file format
CREATE EXTERNAL FILE FORMAT ParquetFormat
WITH (FORMAT_TYPE = PARQUET);

-- Create external table
CREATE EXTERNAL TABLE ExternalSales
WITH (LOCATION = '/sales/', DATA_SOURCE = DataLakeSource, FILE_FORMAT = ParquetFormat);

-- Use COPY statement for fast loading
COPY INTO StagingTable
FROM 'https://account.blob.core.windows.net/container/*.parquet'
WITH (FILE_TYPE = 'PARQUET');

-- Create statistics
CREATE STATISTICS stats_date ON SalesTable(SaleDate);

-- Update statistics
UPDATE STATISTICS SalesTable;
```

---

## 10. Troubleshooting Common Issues

### Data Factory Issues
- **Pipeline failures**: Check activity outputs and diagnostic logs
- **Slow copy performance**: Review DIU settings and parallelism
- **Authentication errors**: Verify linked service credentials and permissions
- **Timeout issues**: Adjust activity timeout settings

### Synapse Analytics Issues
- **Query performance**: Check distribution, indexing, and statistics
- **Concurrency limits**: Review resource class assignments
- **Load failures**: Examine rejected rows and error files
- **Memory errors**: Optimize resource class or simplify queries

### Databricks Issues
- **Cluster startup delays**: Consider pool clusters for faster start
- **Out of memory**: Increase driver/executor memory or reduce data per partition
- **Slow jobs**: Review shuffle operations and data skew
- **Library conflicts**: Isolate dependencies using cluster-scoped libraries

### Stream Analytics Issues
- **Late arrival events**: Adjust late arrival tolerance policy
- **Out-of-order events**: Configure out-of-order tolerance window
- **Insufficient SUs**: Scale streaming units based on query complexity
- **Output errors**: Verify output connection strings and permissions

---

## Appendix: Service Limits and Quotas

### Azure Data Lake Storage Gen2
- Max storage account size: 5 PB
- Max blob size: 190.7 TiB
- Max throughput: 60 Gbps ingress, 120 Gbps egress

### Azure Synapse Analytics
- Max DWU for dedicated pool: 30,000 DWU
- Max concurrent queries (serverless): 20
- Max Spark pools per workspace: 20

### Azure Data Factory
- Max activities per pipeline: 40
- Max parameters per pipeline: 50
- Max integration runtimes per factory: 100
- Max concurrent pipeline runs: 100,000

### Azure Event Hubs
- Max throughput units (Standard): 40
- Max message size: 1 MB
- Max partition count: 32 (Standard), 100 (Premium)
- Max retention period: 7 days (Standard), 90 days (Premium)

### Azure Databricks
- Max clusters per workspace: 150
- Max nodes per cluster: 250
- Max concurrent jobs: 1,000

---

## Conclusion

This fact sheet provides a comprehensive overview of Azure data engineering services and concepts covered in the DP-203 exam. Focus on hands-on practice with these services, understand their capabilities and limitations, and learn when to apply each service to real-world scenarios. Review the linked documentation regularly as Azure services are continuously updated with new features.

**Total Documentation Links: 120**

Good luck with your DP-203 certification exam!
