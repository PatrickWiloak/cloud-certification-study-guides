# Azure Data Engineer Associate (DP-203) Practice Questions

40 scenario-based questions weighted by exam domain.

## Exam Domain Breakdown
| Domain | Weight | Questions |
|--------|--------|-----------|
| Design and implement data storage | 15-20% | 7 |
| Develop data processing | 40-45% | 17 |
| Secure, monitor, and optimize data storage and data processing | 30-35% | 16 |

---

## Domain 1: Design and Implement Data Storage (Questions 1-7)

### Question 1
**Scenario:** A retail company needs to build a data lake to store raw sales data (JSON files), processed data for analytics, and curated data for reporting. They need to support both batch and streaming ingestion with cost-effective storage tiers. What Azure storage architecture should be implemented?

A. Azure Blob Storage with a single container
B. Azure Data Lake Storage Gen2 with bronze/silver/gold zones using hierarchical namespace and lifecycle policies
C. Azure SQL Database for all data
D. Azure Files for storage

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** ADLS Gen2 provides hierarchical namespace (file system semantics) optimized for analytics. Bronze (raw), silver (cleansed), gold (curated) zones follow medallion architecture best practices. Lifecycle policies automatically tier cold data to lower-cost storage. Single container (A) lacks organization. SQL (C) isn't suitable for raw file storage. Azure Files (D) is for SMB scenarios.

**Key Concept:** [Azure Data Lake Storage Gen2](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-introduction)
</details>

### Question 2
**Scenario:** A company needs to store time-series IoT sensor data with high write throughput (millions of records per second) and fast range queries by device ID and time window. They need automatic scaling and global distribution. Which database service is BEST suited?

A. Azure SQL Database
B. Azure Cosmos DB with time series optimized container and partition key on device ID
C. Azure Table Storage
D. Azure Database for PostgreSQL

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Cosmos DB provides unlimited scale with automatic partitioning. Device ID as partition key ensures co-located device data for efficient range queries. Time series containers optimize for append-heavy workloads. Global distribution with multi-region writes supports IoT scale. SQL Database (A) has scaling limits. Table Storage (C) has limited query capabilities. PostgreSQL (D) requires manual sharding at scale.

**Key Concept:** [Cosmos DB for IoT](https://docs.microsoft.com/azure/cosmos-db/use-cases#iot-and-telematics)
</details>

### Question 3
**Scenario:** A media company stores video files in Azure Blob Storage. They need to optimize costs - videos older than 30 days are rarely accessed, and videos older than 1 year are almost never accessed but must be retained for compliance. What storage strategy should be implemented?

A. Store all videos in Hot tier
B. Lifecycle management policy: Hot tier for ≤30 days, Cool tier for 30-365 days, Archive tier for >365 days
C. Delete old videos
D. Store all videos in Archive tier

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Lifecycle management automatically transitions blobs between tiers based on last access or modification time. Hot tier for frequently accessed (higher storage cost, lower access cost). Cool for infrequent access (lower storage, higher access). Archive for rare access (lowest storage, highest access, rehydration delay). Storing all in one tier (A, D) isn't cost-optimized. Deletion (C) violates compliance.

**Key Concept:** [Blob Lifecycle Management](https://docs.microsoft.com/azure/storage/blobs/lifecycle-management-overview)
</details>

### Question 4
**Scenario:** A financial services company needs a relational data warehouse for their BI and reporting workloads. They need to store 50TB of historical data, support concurrent queries from hundreds of analysts, and scale compute independently from storage. What Azure service should they use?

A. Azure SQL Database
B. Azure Synapse Analytics dedicated SQL pool (formerly SQL DW)
C. Azure Database for MySQL
D. Azure Cosmos DB

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Synapse dedicated SQL pools are purpose-built for data warehousing with MPP (massively parallel processing) architecture. Storage and compute scale independently. Supports petabyte-scale data with columnstore indexes for analytical queries. Result caching improves concurrent query performance. SQL Database (A) is OLTP optimized. MySQL (C) isn't designed for DW scale. Cosmos DB (D) is NoSQL.

**Key Concept:** [Synapse Dedicated SQL Pools](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is)
</details>

### Question 5
**Scenario:** A healthcare organization needs to store patient records with strict data isolation requirements. Different hospitals must not be able to access each other's data, but cross-hospital aggregate reports are needed. What data architecture supports this?

A. Single database with application-level filtering
B. Separate storage accounts per hospital with Azure Purview for unified data governance and aggregation pipelines respecting access boundaries
C. Single storage container for all hospitals
D. Completely separate systems with no integration

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Separate storage accounts provide physical data isolation with independent access controls per hospital. Azure Purview provides unified data catalog and governance across accounts. Aggregation pipelines can process data respecting access boundaries, producing aggregate reports without exposing individual hospital data. Single database (A, C) risks cross-hospital exposure. Complete separation (D) prevents needed aggregation.

**Key Concept:** [Azure Purview](https://docs.microsoft.com/azure/purview/overview)
</details>

### Question 6
**Scenario:** A company is migrating a 10TB SQL Server data warehouse to Azure. They need to minimize downtime during migration, validate data integrity, and optimize for Azure's architecture. What migration approach should they use?

A. Backup and restore directly
B. Azure Database Migration Service with online migration, followed by Synapse-specific optimizations (distribution, indexing)
C. Export to CSV and reimport
D. Rebuild from scratch in Azure

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Azure DMS provides online migration with continuous replication, minimizing downtime (cutover only requires catching up recent changes). After migration, optimize for Synapse: choose appropriate distribution (hash for large tables, replicate for small dimension tables), create columnstore indexes, and partition large fact tables. Backup/restore (A) requires extended downtime. CSV export (C) loses schema details. Rebuild (D) is time-consuming.

**Key Concept:** [Azure Database Migration Service](https://docs.microsoft.com/azure/dms/dms-overview)
</details>

### Question 7
**Scenario:** A data lake needs to support both batch analytics (daily reports) and near-real-time analytics (dashboards showing data within 5 minutes of arrival). Data arrives as events through Event Hubs. What architecture pattern enables both use cases?

A. Store all data in relational database
B. Lambda architecture: Stream path (Event Hubs → Stream Analytics → serving layer) and batch path (Event Hubs → Data Lake → Synapse batch processing)
C. Batch processing only with hourly runs
D. Real-time only with no historical analysis

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Lambda architecture maintains separate paths: streaming for low-latency queries (serving layer updated in real-time) and batch for comprehensive historical analysis (corrects any streaming approximations). Both paths serve different query patterns. Event Hubs can feed both paths via capture and streaming. Pure batch (C) can't meet 5-minute SLA. Real-time only (D) loses historical analysis capability.

**Key Concept:** [Lambda Architecture on Azure](https://docs.microsoft.com/azure/architecture/data-guide/big-data/#lambda-architecture)
</details>

---

## Domain 2: Develop Data Processing (Questions 8-24)

### Question 8
**Scenario:** A company needs to process streaming data from IoT devices, perform tumbling window aggregations (5-minute counts per device), and write results to Cosmos DB for real-time dashboards. What Azure service should be used for stream processing?

A. Azure Data Factory
B. Azure Stream Analytics with tumbling window query and Cosmos DB output
C. Azure Batch
D. Azure Logic Apps

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Stream Analytics is designed for real-time stream processing with SQL-like query language supporting windowing functions (tumbling, hopping, sliding, session). Native connectors to IoT Hub/Event Hubs for input and Cosmos DB for output. Fully managed with auto-scaling. Data Factory (A) is for orchestration, not streaming. Batch (C) is for batch workloads. Logic Apps (D) is for workflows, not stream processing.

**Key Concept:** [Stream Analytics Windowing](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-window-functions)
</details>

### Question 9
**Scenario:** A data engineering team needs to build a complex data pipeline that: reads from multiple sources (SQL, APIs, files), applies business logic transformations, handles slowly changing dimensions, and writes to a data warehouse. They want visual development with code generation. What tool should they use?

A. Azure Stream Analytics
B. Azure Synapse Analytics Mapping Data Flows
C. Manual SQL scripting only
D. Azure Functions

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Mapping Data Flows provide visual ETL development with a drag-and-drop interface generating Spark code. Built-in transformations include slowly changing dimension handling, data quality rules, and complex joins. Native integration with multiple sources and Synapse as destination. Stream Analytics (A) is for streaming. Manual SQL (C) lacks visual development. Functions (D) aren't designed for ETL.

**Key Concept:** [Mapping Data Flows](https://docs.microsoft.com/azure/data-factory/concepts-data-flow-overview)
</details>

### Question 10
**Scenario:** A pipeline needs to incrementally load data from an on-premises SQL Server to Azure Data Lake. Only rows modified since the last load should be transferred to minimize data movement. What pattern should be implemented?

A. Full load every time
B. Watermark-based incremental load using Data Factory with change tracking or timestamp column comparison
C. Manual file comparison
D. Trigger-based real-time sync

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Watermark pattern tracks the last processed value (timestamp or ID). Data Factory queries source for records > watermark, loads them, then updates watermark. Change Tracking in SQL Server or modified timestamp columns provide the watermark. This minimizes data transfer. Full load (A) is inefficient for large tables. Manual comparison (C) is error-prone. Real-time sync (D) requires always-on connection.

**Key Concept:** [Incremental Loading](https://docs.microsoft.com/azure/data-factory/tutorial-incremental-copy-overview)
</details>

### Question 11
**Scenario:** A data engineer needs to transform a 500GB dataset using complex Python-based transformations including custom machine learning feature engineering. The processing should scale across multiple nodes. What Azure service and approach should be used?

A. Azure Functions for transformation
B. Azure Databricks or Synapse Spark with PySpark DataFrame operations distributed across cluster
C. Single-node Python script
D. Stream Analytics

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Databricks and Synapse Spark provide distributed processing with PySpark. Operations on DataFrames are automatically parallelized across cluster nodes. Python UDFs enable custom ML feature engineering. Scales to petabytes. Functions (A) have execution time limits and aren't designed for large batch processing. Single-node (C) can't handle 500GB efficiently. Stream Analytics (D) uses SQL, not Python.

**Key Concept:** [Azure Databricks](https://docs.microsoft.com/azure/databricks/scenarios/what-is-azure-databricks)
</details>

### Question 12
**Scenario:** A Spark job processes data from Azure Data Lake Storage Gen2. Performance is slow due to small file problem - the source contains millions of files under 1MB each. How should this be addressed?

A. Process files one at a time
B. Use auto-compaction or run compaction jobs to merge small files into larger files (target 256MB-1GB)
C. Increase cluster size indefinitely
D. Ignore the problem

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Small files create excessive overhead: each file requires separate task, metadata operations, and network calls. Compaction combines small files into optimally-sized files (256MB-1GB for Spark). Delta Lake provides auto-compaction (OPTIMIZE command). This dramatically improves read performance. Processing sequentially (A) doesn't leverage parallelism. More nodes (C) don't solve small file overhead. Ignoring (D) wastes resources.

**Key Concept:** [Delta Lake Optimization](https://docs.microsoft.com/azure/databricks/delta/optimizations/)
</details>

### Question 13
**Scenario:** A data pipeline must ensure exactly-once processing semantics when writing to multiple targets (data lake, data warehouse, operational database). A failure mid-pipeline should not result in partial data in any target. What approach provides this guarantee?

A. Hope failures don't happen
B. Implement transactional writes using Delta Lake with ACID transactions, and database transactions for warehouse writes
C. Write to targets sequentially
D. Retry from beginning on any failure

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Delta Lake provides ACID transactions on data lakes - writes are atomic and either fully succeed or are rolled back. Database transactions provide similar guarantees for warehouse writes. Orchestrate with checkpointing so partial pipeline failure can resume from checkpoint without duplicating completed work. Hope (A) isn't reliable. Sequential writes (C) don't guarantee atomicity. Full retry (D) causes duplicates.

**Key Concept:** [Delta Lake ACID Transactions](https://docs.microsoft.com/azure/databricks/delta/delta-batch)
</details>

### Question 14
**Scenario:** A company needs to orchestrate a data pipeline that runs nightly: first validate source data quality, then run transformations only if validation passes, and send alerts on failure. Dependencies must be handled correctly. What service orchestrates this?

A. Windows Task Scheduler
B. Azure Data Factory with validation activity, conditional paths (if/else), and webhook/email activities for alerts
C. Manual execution
D. Cron on a VM

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Data Factory provides pipeline orchestration with activities, dependencies, and control flow. Validation activities check data quality. If Condition activity branches on success/failure. Web activity or Logic Apps integration sends alerts. Triggers schedule nightly execution. Task Scheduler (A) and cron (D) don't handle Azure-native orchestration with dependencies. Manual execution (C) doesn't scale.

**Key Concept:** [Data Factory Control Flow](https://docs.microsoft.com/azure/data-factory/control-flow-expression-language-functions)
</details>

### Question 15
**Scenario:** A Synapse Spark notebook needs to read data from Azure Key Vault for connection strings and process data from ADLS. The notebook should not contain any hardcoded credentials. How should secrets be accessed?

A. Hardcode credentials in notebook
B. Use linked service with Key Vault connection, or Spark configuration to mount Key Vault-backed secret scope
C. Store credentials in notebook cell
D. Pass credentials as notebook parameters

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Linked services can reference Key Vault secrets for data source credentials. In Databricks/Synapse Spark, secret scopes backed by Key Vault allow retrieving secrets via `dbutils.secrets.get()`. No credentials in code. Synapse workspaces use managed identity to access Key Vault. Hardcoding (A, C) risks exposure in version control. Parameters (D) still expose secrets in pipeline definitions.

**Key Concept:** [Key Vault Linked Service](https://docs.microsoft.com/azure/data-factory/store-credentials-in-key-vault)
</details>

### Question 16
**Scenario:** A data engineer needs to implement slowly changing dimension Type 2 (SCD2) for a customer dimension table. The pipeline should track historical changes with effective dates while maintaining current record flags. What approach is recommended?

A. Overwrite customer records on changes
B. Mapping Data Flow SCD transformation or Delta Lake MERGE with custom SCD2 logic
C. Delete and reload all data daily
D. Ignore historical changes

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Mapping Data Flows have built-in SCD transformation handling Type 1 (overwrite) and Type 2 (history tracking). Alternatively, Delta Lake MERGE with WHEN MATCHED and WHEN NOT MATCHED logic can expire old records (set end date, unset current flag) and insert new versions. This preserves history. Overwrite (A) loses history. Delete/reload (C) is inefficient. Ignoring changes (D) loses valuable temporal data.

**Key Concept:** [SCD in Mapping Data Flows](https://docs.microsoft.com/azure/data-factory/tutorial-data-flow-scd)
</details>

### Question 17
**Scenario:** A company has existing SQL-based ETL logic in stored procedures that they want to migrate to Azure. They want to minimize rewriting while gaining cloud benefits (scale, managed infrastructure). What approach enables this?

A. Completely rewrite in Spark
B. Azure Synapse dedicated SQL pool supporting T-SQL stored procedures with MPP execution
C. Run stored procedures on on-premises SQL Server
D. Convert all logic to Python

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Synapse dedicated SQL pools support T-SQL stored procedures, allowing migration of existing SQL logic with minimal changes. The MPP engine parallelizes execution across distributions. This preserves SQL investment while gaining cloud scale. Complete rewrite (A, D) requires significant effort. On-premises (C) doesn't gain cloud benefits.

**Key Concept:** [Synapse T-SQL Support](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-develop-stored-procedures)
</details>

### Question 18
**Scenario:** A data pipeline reads JSON files with a schema that changes frequently (new fields added). The pipeline should automatically handle schema evolution without manual intervention or failures. What storage format and approach should be used?

A. Fixed schema relational tables
B. Delta Lake with schema evolution enabled (mergeSchema option) allowing automatic schema updates
C. Fail on schema mismatch
D. Pre-define all possible fields

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Delta Lake with `mergeSchema=true` automatically adds new columns when they appear in source data. This handles additive schema evolution without pipeline changes. Schema enforcement prevents incompatible changes (type mismatches). Fixed schemas (A) require manual updates. Failing (C) causes pipeline disruptions. Pre-defining (D) requires knowing future schema changes.

**Key Concept:** [Delta Lake Schema Evolution](https://docs.microsoft.com/azure/databricks/delta/delta-batch#schema-evolution)
</details>

### Question 19
**Scenario:** A Spark job needs to join a 500GB fact table with a 50MB dimension table. The dimension table easily fits in memory on each worker. How should this join be optimized?

A. Shuffle hash join
B. Broadcast join (broadcast the dimension table to all workers)
C. Sort merge join
D. Nested loop join

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Broadcast join sends the small table to all workers, enabling local joins without shuffling the large table. Since 50MB easily fits in memory, this eliminates expensive network shuffle of the 500GB table. Use broadcast hint: `fact_df.join(broadcast(dim_df), ...)`. Shuffle joins (A, C) move data unnecessarily. Nested loop (D) is extremely slow.

**Key Concept:** [Spark Broadcast Joins](https://spark.apache.org/docs/latest/sql-performance-tuning.html#broadcast-hint-for-sql-queries)
</details>

### Question 20
**Scenario:** A real-time pipeline processes user clickstream events and needs to detect user sessions (group of events within 30 minutes of inactivity). Events should be grouped by user with session boundaries. What windowing approach supports this?

A. Tumbling window
B. Session window with 30-minute gap duration grouped by user ID
C. Sliding window
D. Hopping window

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Session windows group events with dynamic boundaries based on inactivity gaps. A 30-minute gap means a new session starts after 30 minutes without events from that user. This naturally models user sessions without fixed window sizes. Tumbling (A) and hopping (D) have fixed sizes. Sliding (C) overlaps but doesn't detect gaps.

**Key Concept:** [Stream Analytics Session Windows](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-window-functions#session-window)
</details>

### Question 21
**Scenario:** A Synapse Spark notebook processes data and results should be available for T-SQL queries in Synapse serverless SQL pool. How can Spark write data that's queryable via SQL without copying to dedicated pool?

A. Copy data to dedicated SQL pool
B. Write Spark output as Delta Lake or Parquet to ADLS, then query via serverless SQL pool with OPENROWSET
C. Create JDBC connection from SQL to Spark
D. Export to CSV for manual import

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Synapse serverless SQL pool can query files in ADLS directly using OPENROWSET. Spark writes Delta/Parquet to ADLS, then T-SQL queries same files. No data copying - single copy of data serves both Spark and SQL queries. Delta provides ACID transactions visible to both. Copying (A) duplicates data. JDBC (C) isn't the pattern. CSV export (D) loses performance and schema.

**Key Concept:** [Serverless SQL Pool](https://docs.microsoft.com/azure/synapse-analytics/sql/on-demand-workspace-overview)
</details>

### Question 22
**Scenario:** A data pipeline needs to handle late-arriving data - events can arrive up to 24 hours after their actual occurrence time. Downstream aggregations should use event time, not processing time, and incorporate late data. What approach handles this?

A. Ignore late data
B. Use event time watermarking with 24-hour allowed lateness, enabling Spark Structured Streaming or Stream Analytics to include late events
C. Process only data that arrives on time
D. Reprocess entire dataset when late data arrives

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Watermarking tells the streaming engine how late data can arrive. Event time processing uses timestamps from data, not arrival time. Late events within the watermark threshold are incorporated into correct time windows. Events beyond threshold are dropped (configurable). Ignoring (A, C) loses data. Full reprocess (D) is expensive and doesn't support streaming.

**Key Concept:** [Structured Streaming Watermarking](https://spark.apache.org/docs/latest/structured-streaming-programming-guide.html#handling-late-data-and-watermarking)
</details>

### Question 23
**Scenario:** A company needs to implement data lineage tracking - understanding where data comes from, how it's transformed, and where it goes. This is required for regulatory compliance and impact analysis. What Azure service provides this?

A. Manual documentation
B. Microsoft Purview (Azure Purview) with automated lineage capture from Data Factory, Synapse, and Databricks
C. Source code review
D. Spreadsheet tracking

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Purview automatically captures lineage from Data Factory pipelines, Synapse notebooks, and Databricks jobs. It shows the data flow from source to destination through transformations. Enables impact analysis (what's affected if source changes) and root cause analysis (where did bad data originate). Manual methods (A, C, D) don't scale and become outdated.

**Key Concept:** [Purview Data Lineage](https://docs.microsoft.com/azure/purview/concept-data-lineage)
</details>

### Question 24
**Scenario:** A pipeline processes 1TB of data daily. Currently, it runs for 4 hours using 10 worker nodes. The team needs to reduce runtime to under 1 hour without changing the code logic. What approaches can improve performance?

A. Accept 4-hour runtime
B. Scale out to more nodes, optimize Spark configurations (partition count, memory settings), and add caching for reused DataFrames
C. Add more data to process
D. Use single-threaded processing

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Scale out (more workers) enables more parallelism. Optimize partitions (repartition for better distribution, coalesce to reduce shuffle). Tune memory (spark.executor.memory) based on workload. Cache DataFrames that are accessed multiple times. These can reduce runtime significantly. Accepting slow runtime (A) doesn't meet requirement. More data (C) worsens runtime. Single-threaded (D) is slower.

**Key Concept:** [Spark Performance Tuning](https://spark.apache.org/docs/latest/tuning.html)
</details>

---

## Domain 3: Secure, Monitor, and Optimize Data Storage and Data Processing (Questions 25-40)

### Question 25
**Scenario:** A data lake contains sensitive customer PII that should only be accessible by specific data teams. Access should be controlled at the file and folder level, and all access should be audited. What security approach should be implemented?

A. Single shared access key for everyone
B. Azure Active Directory authentication with ADLS Gen2 ACLs and Azure RBAC for fine-grained access control
C. Public access with IP filtering
D. Shared passwords

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** AAD authentication provides identity-based access (not shared keys). ADLS Gen2 ACLs enable POSIX-like permissions at folder/file level. Azure RBAC provides role-based access at broader scopes. All access is logged for audit. Shared keys (A, D) don't provide user attribution. Public access (C) exposes data regardless of IP filtering risks.

**Key Concept:** [ADLS Gen2 Access Control](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-access-control)
</details>

### Question 26
**Scenario:** A Synapse dedicated SQL pool contains sensitive columns (SSN, credit card numbers) that most analysts should not see. Only authorized analysts should access these columns. How should column-level security be implemented?

A. Remove sensitive columns from the table
B. Dynamic Data Masking to obfuscate sensitive columns for unauthorized users, with column-level permissions via GRANT/DENY
C. Trust users not to query sensitive columns
D. Create separate tables for sensitive data

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Dynamic Data Masking obfuscates data at query time for users without unmask permission - they see masked values (e.g., XXXX-XXXX-XXXX-1234 for credit cards). GRANT/DENY at column level provides fine-grained access control. Authorized users see full data. Removing columns (A) loses data. Trust (C) isn't enforcement. Separate tables (D) complicate queries.

**Key Concept:** [Dynamic Data Masking](https://docs.microsoft.com/azure/azure-sql/database/dynamic-data-masking-overview)
</details>

### Question 27
**Scenario:** A company needs to encrypt all data at rest in their Azure Data Lake Storage Gen2 account. They want to manage their own encryption keys and rotate them annually. How should this be configured?

A. Rely on default Microsoft-managed encryption
B. Customer-managed keys in Azure Key Vault with automatic rotation configured for the storage account
C. Client-side encryption only
D. No encryption

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** ADLS Gen2 supports customer-managed keys (CMK) stored in Azure Key Vault. Configure the storage account to use Key Vault key for encryption. Key Vault supports automatic key rotation while maintaining access to data encrypted with previous versions. Default encryption (A) doesn't allow customer key control. Client-side (C) adds application complexity. No encryption (D) violates security requirements.

**Key Concept:** [Customer-Managed Keys for Storage](https://docs.microsoft.com/azure/storage/common/customer-managed-keys-overview)
</details>

### Question 28
**Scenario:** A data pipeline processes sensitive healthcare data. Regulations require that the data is encrypted not only at rest but also in transit between all components (storage, processing, warehouse). How should in-transit encryption be ensured?

A. Assume encryption is handled automatically
B. Enforce HTTPS-only access for storage accounts, enable encrypted connections for SQL pools, and use encrypted Spark shuffle
C. Use HTTP for performance
D. Encrypt at rest is sufficient

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Storage account "secure transfer required" enforces HTTPS. Synapse/SQL connections use TLS by default but should be verified. Spark shuffle encryption protects data moving between executors. Together these ensure data is encrypted in transit at all stages. Assumptions (A) are risky. HTTP (C) transmits in clear text. At-rest only (D) leaves transit vulnerable.

**Key Concept:** [Secure Transfer Required](https://docs.microsoft.com/azure/storage/common/storage-require-secure-transfer)
</details>

### Question 29
**Scenario:** A Spark job is failing intermittently with out-of-memory errors. Monitoring shows some executors using much more memory than others. The job performs aggregations on skewed data where some keys have millions of rows while others have few. How should this be addressed?

A. Add more memory uniformly to all executors
B. Address data skew using salting technique (add random prefix to skewed keys, aggregate, then remove prefix)
C. Reduce parallelism
D. Accept failures and retry

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Data skew causes some partitions to have disproportionate data. Salting distributes hot keys across partitions by adding random prefix, enabling parallel processing. After initial aggregation, remove salt for final aggregation. This balances work. Uniform memory increase (A) doesn't fix skew. Reduced parallelism (C) worsens the problem. Retries (D) don't address root cause.

**Key Concept:** [Handling Data Skew](https://docs.microsoft.com/azure/databricks/kb/sql/handle-skewed-data)
</details>

### Question 30
**Scenario:** A data warehouse query that previously ran in 30 seconds now takes 5 minutes after data volume increased. The query joins a large fact table with several dimension tables. How should query performance be diagnosed and improved?

A. Accept slower performance
B. Analyze query execution plan, check distribution strategy (hash-distribute large fact table on join key), ensure statistics are up to date
C. Add more DWUs without investigation
D. Rewrite query in different language

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Execution plan reveals if data movement (shuffle) is causing slowdown. Hash distribution on join key co-locates data for efficient joins. Replicate small dimension tables to eliminate broadcast. Update statistics helps optimizer choose better plans. More DWUs (C) may help but without diagnosing the issue, it's inefficient. Accepting slowdown (A) affects users. Language change (D) doesn't address distribution.

**Key Concept:** [Synapse Distribution Guidance](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/sql-data-warehouse-tables-distribute)
</details>

### Question 31
**Scenario:** A Data Factory pipeline runs daily and occasionally fails due to transient issues (network timeouts, service throttling). The team wants automatic retries before alerting. How should retry behavior be configured?

A. Manual monitoring and re-triggering
B. Configure activity retry policy with count and interval, and pipeline retry on failure
C. No retries - fail immediately
D. Infinite retries

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Activity retry policy specifies how many times to retry (e.g., 3) and interval between retries (e.g., 30 seconds). This handles transient failures automatically. If all retries fail, pipeline fails and alerting triggers. Manual intervention (A) is inefficient. No retries (C) cause unnecessary failures. Infinite retries (D) can mask persistent problems.

**Key Concept:** [Data Factory Retry Policy](https://docs.microsoft.com/azure/data-factory/concepts-pipelines-activities#activity-retry)
</details>

### Question 32
**Scenario:** A company needs to monitor data quality in their data lake. They want to detect issues like null values in required fields, duplicate records, and values outside expected ranges. Alerts should trigger when quality thresholds are breached. What approach should be implemented?

A. Manual data inspection
B. Implement data quality rules using Delta Live Tables expectations or Great Expectations library, with metrics sent to Azure Monitor
C. Assume data is always clean
D. Quality checks in downstream reports only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Data quality frameworks (Delta Live Tables expectations, Great Expectations) define rules (null checks, uniqueness, ranges) and validate data during processing. Failed expectations can quarantine bad records or fail pipeline. Metrics integrate with Azure Monitor for alerting. Manual inspection (A) doesn't scale. Assumptions (C) lead to bad data. Downstream checks (D) are too late.

**Key Concept:** [Delta Live Tables Expectations](https://docs.microsoft.com/azure/databricks/workflows/delta-live-tables/delta-live-tables-expectations)
</details>

### Question 33
**Scenario:** A Synapse dedicated SQL pool is experiencing high storage costs. Analysis shows that 80% of data is rarely queried historical data, but queries against it must still be supported. How should costs be optimized?

A. Delete historical data
B. Offload cold data to ADLS with external tables (Polybase) maintaining query access while reducing dedicated pool storage
C. Ignore storage costs
D. Compress data only

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** External tables allow querying data in ADLS directly from Synapse SQL without loading into dedicated pool. Move cold historical data to cheaper ADLS storage (with lifecycle management to archive tier). Queries work transparently against external tables. This reduces expensive DW storage. Deletion (A) loses data. Compression (D) helps but doesn't address storage tier costs.

**Key Concept:** [Synapse External Tables](https://docs.microsoft.com/azure/synapse-analytics/sql/develop-tables-external-tables)
</details>

### Question 34
**Scenario:** A streaming pipeline processes financial transactions and must guarantee no data loss even if the processing system fails. Events come from Event Hubs. How should reliability be ensured?

A. Hope failures don't occur
B. Enable Event Hubs Capture for automatic backup to storage, use checkpointing in Stream Analytics/Spark for exactly-once processing
C. Process messages without acknowledgment
D. Single consumer without failover

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Event Hubs Capture automatically stores all events to ADLS/Blob as backup. Checkpointing tracks processing progress - on restart, processing resumes from checkpoint. This enables exactly-once semantics and recovery from failures. Hope (A) is not a strategy. No acknowledgment (C) causes data loss. Single consumer (D) has no redundancy.

**Key Concept:** [Event Hubs Capture](https://docs.microsoft.com/azure/event-hubs/event-hubs-capture-overview)
</details>

### Question 35
**Scenario:** A Databricks cluster is significantly underutilized - jobs run for 2 hours but 80% of nodes are idle for most of the time due to variable workload stages. How should cluster efficiency be improved?

A. Use fixed-size clusters
B. Enable cluster autoscaling with appropriate min/max workers to scale down during low-utilization phases
C. Use larger node types
D. Run jobs sequentially

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Autoscaling dynamically adjusts cluster size based on workload. Set minimum for baseline and maximum for peak. Cluster scales down during low-utilization stages, reducing cost. Scales up when workload increases. Fixed size (A) wastes resources during idle periods. Larger nodes (C) don't help if they're underutilized. Sequential jobs (D) extend total runtime.

**Key Concept:** [Databricks Autoscaling](https://docs.microsoft.com/azure/databricks/clusters/cluster-config-best-practices#cluster-sizing)
</details>

### Question 36
**Scenario:** A Data Factory pipeline moves data from on-premises SQL Server to Azure Data Lake. Network bandwidth between on-premises and Azure is limited, causing slow transfers. How can transfer performance be improved?

A. Accept slow transfers
B. Use Self-Hosted Integration Runtime with compression enabled, and consider parallelizing with multiple copy activities
C. Transfer over public internet without optimization
D. Send physical media

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Self-Hosted IR runs on-premises for secure access. Enabling compression reduces bytes transferred over the network. Parallel copy activities (partitioning large tables) utilize bandwidth more efficiently. Staged copy through Azure Storage can also help. Accepting slow transfers (A) affects SLAs. Unoptimized transfer (C) wastes bandwidth. Physical media (D) adds days of latency.

**Key Concept:** [Copy Activity Performance](https://docs.microsoft.com/azure/data-factory/copy-activity-performance)
</details>

### Question 37
**Scenario:** A company needs to demonstrate to auditors that their data pipelines are processing data correctly and that outputs are traceable to inputs. They need comprehensive logging of all pipeline operations. What should be implemented?

A. No logging
B. Enable Data Factory diagnostic logs to Log Analytics, implement logging in Spark notebooks, and use Purview for lineage
C. Console output only
D. Annual manual audits

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Data Factory diagnostic logs capture pipeline runs, activity details, and errors. Log Analytics enables querying and retention. Spark notebooks can log transformations and row counts. Purview provides automated lineage showing data flow from source to destination. Together these provide audit trail. No logging (A) provides no evidence. Console output (C) is lost. Annual audits (D) don't provide continuous evidence.

**Key Concept:** [Data Factory Monitoring](https://docs.microsoft.com/azure/data-factory/monitor-using-azure-monitor)
</details>

### Question 38
**Scenario:** A Stream Analytics job needs to process events with guaranteed ordering per device. Events from each device should be processed in the order they were generated, identified by device ID. How should this be configured?

A. Process all events in any order
B. Use PARTITION BY deviceId in the query, ensuring events from each device are processed in order within their partition
C. Single-threaded processing
D. Sort events after processing

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** PARTITION BY ensures events with the same partition key are processed in order and by the same compute node. Events are ordered within their partition. Different devices can be processed in parallel on different nodes. Any order (A) doesn't meet requirements. Single-threaded (C) doesn't scale. Post-processing sort (D) is too late if processing depends on order.

**Key Concept:** [Stream Analytics Partitioning](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-parallelization)
</details>

### Question 39
**Scenario:** A company uses Synapse serverless SQL pool for ad-hoc queries on data lake files. Query costs are unpredictable and sometimes spike unexpectedly. How should costs be controlled?

A. Unlimited spending
B. Configure cost control settings to limit data processed per query and per day/week
C. Don't use serverless pool
D. Review bills monthly

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Serverless SQL pool cost controls allow setting limits on data processed per query and cumulative limits per time period. Queries exceeding limits are rejected. This prevents runaway costs from poorly written queries or unexpected usage. Unlimited spending (A) risks budget overruns. Avoiding serverless (C) loses flexibility. Monthly review (D) is too late to control costs.

**Key Concept:** [Serverless SQL Cost Control](https://docs.microsoft.com/azure/synapse-analytics/sql/data-processed)
</details>

### Question 40
**Scenario:** A data engineering team needs to implement row-level security in Azure Synapse dedicated SQL pool. Sales managers should only see data for their region when querying a shared sales fact table. How should this be implemented?

A. Create separate tables per region
B. Implement Row-Level Security (RLS) with security predicate filtering rows based on user's region assignment
C. Application-level filtering only
D. Trust users to add WHERE clauses

<details>
<summary>Answer</summary>

**Correct: B**

**Why:** Row-Level Security creates a security predicate function that filters rows based on user context (e.g., SUSER_SNAME() mapped to region). Applied as security policy on the table, it transparently filters all queries. Users automatically see only their data. Separate tables (A) complicate queries and maintenance. Application filtering (C) can be bypassed with direct SQL access. Trust (D) isn't enforcement.

**Key Concept:** [Row-Level Security](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/column-level-security)
</details>
