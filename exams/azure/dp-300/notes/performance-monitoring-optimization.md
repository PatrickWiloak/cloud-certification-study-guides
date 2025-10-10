# Performance Monitoring and Optimization

## Table of Contents
1. [Performance Monitoring Tools](#performance-monitoring-tools)
2. [Query Performance Analysis](#query-performance-analysis)
3. [Index Optimization](#index-optimization)
4. [Query Tuning Techniques](#query-tuning-techniques)
5. [Resource Monitoring and Management](#resource-monitoring-and-management)
6. [Troubleshooting Performance Issues](#troubleshooting-performance-issues)
7. [Automatic Tuning](#automatic-tuning)
8. [Exam Tips](#exam-tips)

---

## Performance Monitoring Tools

### Azure Monitor

#### Metrics
**Key Performance Metrics:**
- CPU percentage
- Data IO percentage
- Log IO percentage
- DTU/vCore percentage (SQL Database)
- Storage percentage
- Connection counts
- Deadlocks
- Blocked queries

**Metric Configuration:**
- Real-time monitoring
- Historical data (93 days)
- Custom time ranges
- Aggregation options (avg, min, max, sum)

**Alerts:**
```powershell
# Create metric alert
$criteria = New-AzMetricAlertRuleV2Criteria `
    -MetricName "cpu_percent" `
    -TimeAggregation Average `
    -Operator GreaterThan `
    -Threshold 80

New-AzMetricAlertRuleV2 `
    -Name "HighCPUAlert" `
    -ResourceGroupName "myRG" `
    -WindowSize (New-TimeSpan -Minutes 5) `
    -Frequency (New-TimeSpan -Minutes 1) `
    -TargetResourceId "/subscriptions/.../databases/mydb" `
    -Condition $criteria `
    -ActionGroupId "/subscriptions/.../actionGroups/myActionGroup" `
    -Severity 2
```

#### Activity Log
- Administrative operations
- Service health events
- Resource changes
- Deployment history

#### Diagnostic Logs
**Categories:**
- SQLInsights
- AutomaticTuning
- QueryStoreRuntimeStatistics
- QueryStoreWaitStatistics
- Errors
- DatabaseWaitStatistics
- Timeouts
- Blocks
- Deadlocks

**Enable Diagnostics:**
```powershell
Set-AzDiagnosticSetting `
    -ResourceId "/subscriptions/.../databases/mydb" `
    -Name "DiagSettings" `
    -Enabled $true `
    -WorkspaceId "/subscriptions/.../workspaces/myworkspace" `
    -Category @("QueryStoreRuntimeStatistics", "Errors", "Deadlocks")
```

### Azure SQL Analytics
Log Analytics solution for SQL performance.

**Features:**
- Cross-database monitoring
- Performance dashboards
- Intelligent insights
- Query performance trends
- Resource utilization
- Wait statistics

**Setup:**
1. Create Log Analytics workspace
2. Add Azure SQL Analytics solution
3. Configure diagnostic settings
4. View dashboards and reports

**Key Dashboards:**
- Performance overview
- Resource consumption
- Query performance
- Intelligent Insights
- Wait statistics

### Query Store
Built-in query performance monitoring.

#### Features
- Automatic query capture
- Execution plans
- Runtime statistics
- Historical performance
- Query regression detection
- Force plans

#### Configuration
```sql
-- Enable Query Store
ALTER DATABASE [MyDB]
SET QUERY_STORE = ON
(
    OPERATION_MODE = READ_WRITE,
    CLEANUP_POLICY = (STALE_QUERY_THRESHOLD_DAYS = 30),
    DATA_FLUSH_INTERVAL_SECONDS = 900,
    INTERVAL_LENGTH_MINUTES = 60,
    MAX_STORAGE_SIZE_MB = 100,
    QUERY_CAPTURE_MODE = AUTO,
    SIZE_BASED_CLEANUP_MODE = AUTO
);

-- Check Query Store status
SELECT * FROM sys.database_query_store_options;
```

#### Query Store Views
```sql
-- Top resource consuming queries
SELECT TOP 10
    qsq.query_id,
    qsqt.query_sql_text,
    rs.count_executions,
    rs.avg_duration / 1000.0 AS avg_duration_ms,
    rs.avg_cpu_time / 1000.0 AS avg_cpu_time_ms,
    rs.avg_logical_io_reads
FROM sys.query_store_query qsq
JOIN sys.query_store_query_text qsqt ON qsq.query_text_id = qsqt.query_text_id
JOIN sys.query_store_plan qsp ON qsq.query_id = qsp.query_id
JOIN sys.query_store_runtime_stats rs ON qsp.plan_id = rs.plan_id
WHERE rs.last_execution_time >= DATEADD(hour, -1, GETUTCDATE())
ORDER BY rs.avg_duration DESC;

-- Query plan regression
SELECT
    qsq.query_id,
    qsqt.query_sql_text,
    qsp.plan_id,
    rs.count_executions,
    rs.avg_duration,
    rs.last_execution_time
FROM sys.query_store_query qsq
JOIN sys.query_store_query_text qsqt ON qsq.query_text_id = qsqt.query_text_id
JOIN sys.query_store_plan qsp ON qsq.query_id = qsp.query_id
JOIN sys.query_store_runtime_stats rs ON qsp.plan_id = rs.plan_id
WHERE qsq.query_id IN (
    -- Queries with performance regression
    SELECT query_id FROM sys.query_store_query
    WHERE is_internal_query = 0
);

-- Force execution plan
EXEC sp_query_store_force_plan @query_id = 123, @plan_id = 456;

-- Unforce plan
EXEC sp_query_store_unforce_plan @query_id = 123, @plan_id = 456;
```

### Dynamic Management Views (DMVs)

#### Key DMVs for Performance

**sys.dm_exec_query_stats:**
```sql
-- Top CPU consuming queries
SELECT TOP 20
    qs.total_worker_time / qs.execution_count / 1000.0 AS avg_cpu_ms,
    qs.total_worker_time / 1000.0 AS total_cpu_ms,
    qs.execution_count,
    SUBSTRING(qt.text, qs.statement_start_offset/2 + 1,
        (CASE WHEN qs.statement_end_offset = -1
            THEN LEN(CONVERT(nvarchar(max), qt.text)) * 2
            ELSE qs.statement_end_offset
        END - qs.statement_start_offset)/2 + 1) AS query_text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) qt
ORDER BY qs.total_worker_time DESC;
```

**sys.dm_db_index_usage_stats:**
```sql
-- Unused indexes
SELECT
    OBJECT_NAME(ius.object_id) AS table_name,
    i.name AS index_name,
    ius.user_seeks,
    ius.user_scans,
    ius.user_lookups,
    ius.user_updates
FROM sys.dm_db_index_usage_stats ius
JOIN sys.indexes i ON ius.object_id = i.object_id AND ius.index_id = i.index_id
WHERE ius.database_id = DB_ID()
    AND ius.user_seeks = 0
    AND ius.user_scans = 0
    AND ius.user_lookups = 0
    AND i.type_desc <> 'CLUSTERED'
ORDER BY ius.user_updates DESC;
```

**sys.dm_os_wait_stats:**
```sql
-- Wait statistics
SELECT TOP 20
    wait_type,
    wait_time_ms / 1000.0 AS wait_time_sec,
    (wait_time_ms - signal_wait_time_ms) / 1000.0 AS resource_wait_sec,
    signal_wait_time_ms / 1000.0 AS signal_wait_sec,
    waiting_tasks_count,
    wait_time_ms / waiting_tasks_count AS avg_wait_ms
FROM sys.dm_os_wait_stats
WHERE wait_type NOT IN (
    'BROKER_EVENTHANDLER', 'BROKER_RECEIVE_WAITFOR',
    'BROKER_TASK_STOP', 'BROKER_TO_FLUSH',
    'CHECKPOINT_QUEUE', 'CHKPT', 'CLR_AUTO_EVENT',
    'CLR_MANUAL_EVENT', 'CLR_SEMAPHORE', 'DBMIRROR_DBM_MUTEX',
    'DBMIRROR_EVENTS_QUEUE', 'DBMIRRORING_CMD', 'DIRTY_PAGE_POLL',
    'DISPATCHER_QUEUE_SEMAPHORE', 'EXECSYNC', 'FSAGENT',
    'FT_IFTS_SCHEDULER_IDLE_WAIT', 'FT_IFTSHC_MUTEX',
    'HADR_CLUSAPI_CALL', 'HADR_FILESTREAM_IOMGR_IOCOMPLETION',
    'HADR_LOGCAPTURE_WAIT', 'HADR_NOTIFICATION_DEQUEUE',
    'HADR_TIMER_TASK', 'HADR_WORK_QUEUE',
    'LAZYWRITER_SLEEP', 'LOGMGR_QUEUE',
    'ONDEMAND_TASK_QUEUE', 'PWAIT_ALL_COMPONENTS_INITIALIZED',
    'REQUEST_FOR_DEADLOCK_SEARCH', 'RESOURCE_QUEUE',
    'SERVER_IDLE_CHECK', 'SLEEP_BPOOL_FLUSH', 'SLEEP_DBSTARTUP',
    'SLEEP_DCOMSTARTUP', 'SLEEP_MASTERDBREADY', 'SLEEP_MASTERMDREADY',
    'SLEEP_MASTERUPGRADED', 'SLEEP_MSDBSTARTUP', 'SLEEP_SYSTEMTASK',
    'SLEEP_TASK', 'SLEEP_TEMPDBSTARTUP', 'SNI_HTTP_ACCEPT',
    'SP_SERVER_DIAGNOSTICS_SLEEP', 'SQLTRACE_BUFFER_FLUSH',
    'SQLTRACE_INCREMENTAL_FLUSH_SLEEP', 'SQLTRACE_WAIT_ENTRIES',
    'WAIT_FOR_RESULTS', 'WAITFOR', 'WAITFOR_TASKSHUTDOWN',
    'WAIT_XTP_HOST_WAIT', 'WAIT_XTP_OFFLINE_CKPT_NEW_LOG',
    'WAIT_XTP_CKPT_CLOSE', 'XE_DISPATCHER_JOIN',
    'XE_DISPATCHER_WAIT', 'XE_TIMER_EVENT'
)
ORDER BY wait_time_ms DESC;
```

**sys.dm_exec_requests:**
```sql
-- Currently executing queries
SELECT
    r.session_id,
    r.status,
    r.command,
    r.cpu_time,
    r.total_elapsed_time,
    r.reads,
    r.writes,
    r.wait_type,
    r.wait_time,
    SUBSTRING(qt.text, r.statement_start_offset/2 + 1,
        (CASE WHEN r.statement_end_offset = -1
            THEN LEN(CONVERT(nvarchar(max), qt.text)) * 2
            ELSE r.statement_end_offset
        END - r.statement_start_offset)/2 + 1) AS query_text
FROM sys.dm_exec_requests r
CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) qt
WHERE r.session_id > 50
    AND r.session_id <> @@SPID
ORDER BY r.total_elapsed_time DESC;
```

**sys.dm_db_missing_index_details:**
```sql
-- Missing index recommendations
SELECT
    CONVERT(decimal(18,2), migs.user_seeks * migs.avg_total_user_cost * (migs.avg_user_impact * 0.01)) AS improvement_measure,
    'CREATE INDEX ix_' + OBJECT_NAME(mid.object_id, mid.database_id) + '_'
        + REPLACE(REPLACE(REPLACE(ISNULL(mid.equality_columns,''), ', ', '_'), '[', ''), ']', '')
        + CASE WHEN mid.inequality_columns IS NOT NULL THEN '_' + REPLACE(REPLACE(REPLACE(mid.inequality_columns, ', ', '_'), '[', ''), ']', '') ELSE '' END
        + ' ON ' + mid.statement
        + ' (' + ISNULL(mid.equality_columns,'')
        + CASE WHEN mid.equality_columns IS NOT NULL AND mid.inequality_columns IS NOT NULL THEN ',' ELSE '' END
        + ISNULL(mid.inequality_columns, '') + ')'
        + ISNULL(' INCLUDE (' + mid.included_columns + ')', '') AS create_index_statement,
    migs.user_seeks,
    migs.avg_total_user_cost,
    migs.avg_user_impact
FROM sys.dm_db_missing_index_groups mig
JOIN sys.dm_db_missing_index_group_stats migs ON mig.index_group_handle = migs.group_handle
JOIN sys.dm_db_missing_index_details mid ON mig.index_handle = mid.index_handle
WHERE migs.user_seeks * migs.avg_total_user_cost * (migs.avg_user_impact * 0.01) > 10
    AND mid.database_id = DB_ID()
ORDER BY improvement_measure DESC;
```

### Intelligent Insights
AI-powered performance diagnostics.

**Detects:**
- Performance degradation
- Query regression
- Parameter sensitivity
- Database/elastic pool reaching limits
- Increased wait times
- Excessive locking
- Increased tempdb usage

**Access:**
- Azure portal
- Log Analytics
- Diagnostic logs
- REST API

**Example Query (Log Analytics):**
```kusto
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.SQL"
| where Category == "SQLInsights"
| where status_s == "Active"
| project TimeGenerated, impact_s, rootCauseAnalysis_s, recommendedAction_s
| order by TimeGenerated desc
```

---

## Query Performance Analysis

### Execution Plans

#### Actual vs Estimated Plans

**Estimated Execution Plan:**
- Generated without executing query
- Shows planned operations
- Quick to obtain
- May differ from actual

**Actual Execution Plan:**
- Generated during query execution
- Shows actual operations
- Real row counts
- Actual execution time

#### Reading Execution Plans

**Key Components:**
1. **Operators** - Operations performed
2. **Flow Direction** - Right to left, top to bottom
3. **Cost** - Relative cost percentages
4. **Row Counts** - Estimated vs actual
5. **Warnings** - Yellow exclamation marks

**Common Operators:**
- **Table Scan** - Reads entire table (costly)
- **Index Scan** - Reads entire index
- **Index Seek** - Targeted index lookup (efficient)
- **Clustered Index Scan** - Reads entire table via clustered index
- **Key Lookup** - Retrieve columns not in nonclustered index
- **Nested Loops** - Join algorithm
- **Hash Match** - Join/aggregate algorithm
- **Sort** - Order by operation
- **Filter** - Where clause filtering

**Get Execution Plan in T-SQL:**
```sql
-- Estimated plan
SET SHOWPLAN_XML ON;
GO
SELECT * FROM Orders WHERE OrderDate > '2023-01-01';
GO
SET SHOWPLAN_XML OFF;
GO

-- Actual plan
SET STATISTICS XML ON;
GO
SELECT * FROM Orders WHERE OrderDate > '2023-01-01';
GO
SET STATISTICS XML OFF;
GO

-- Include actual plan in SSMS
-- Ctrl+M or Query -> Include Actual Execution Plan
```

#### Plan Warnings

**Common Warnings:**
- **Missing Index** - Potential index to create
- **Implicit Conversion** - Data type mismatch
- **Missing Statistics** - Need to update stats
- **Spill to TempDB** - Memory grant too small
- **Unmatched Indexes** - Seek predicate warnings

### Query Execution Statistics

```sql
-- IO statistics
SET STATISTICS IO ON;
GO
SELECT * FROM Orders WHERE OrderDate > '2023-01-01';
GO
SET STATISTICS IO OFF;
GO
-- Shows: logical reads, physical reads, read-ahead reads

-- Time statistics
SET STATISTICS TIME ON;
GO
SELECT * FROM Orders WHERE OrderDate > '2023-01-01';
GO
SET STATISTICS TIME OFF;
GO
-- Shows: CPU time, elapsed time

-- Client statistics in SSMS
-- Query -> Include Client Statistics
```

### Query Plan Caching

**Plan Cache DMVs:**
```sql
-- View cached plans
SELECT
    cp.objtype,
    cp.cacheobjtype,
    cp.size_in_bytes / 1024 AS size_kb,
    cp.usecounts,
    qt.text
FROM sys.dm_exec_cached_plans cp
CROSS APPLY sys.dm_exec_sql_text(cp.plan_handle) qt
ORDER BY cp.usecounts DESC;

-- Clear plan cache (use carefully)
DBCC FREEPROCCACHE; -- All plans
DBCC FREEPROCCACHE (plan_handle); -- Specific plan
ALTER DATABASE SCOPED CONFIGURATION CLEAR PROCEDURE_CACHE; -- Database scope
```

**Plan Cache Considerations:**
- Parameterized queries cache better
- Ad-hoc queries can bloat cache
- Plan recompilation overhead
- Statistics updates can invalidate plans

---

## Index Optimization

### Index Types

#### Clustered Index
- Physical order of table
- One per table
- Leaf level contains actual data
- Typically on primary key

**Create:**
```sql
CREATE CLUSTERED INDEX ix_Orders_OrderDate
ON Orders(OrderDate);
```

#### Nonclustered Index
- Separate structure from table
- Multiple per table (999 max)
- Leaf level contains key + bookmark
- Includes covering columns

**Create:**
```sql
CREATE NONCLUSTERED INDEX ix_Orders_CustomerID
ON Orders(CustomerID)
INCLUDE (OrderDate, TotalAmount);
```

#### Columnstore Index
- Columnar storage format
- Excellent compression
- Analytical queries
- Clustered or nonclustered

**Create:**
```sql
-- Nonclustered columnstore (OLTP + analytics)
CREATE NONCLUSTERED COLUMNSTORE INDEX ix_Orders_Columnstore
ON Orders (OrderDate, CustomerID, TotalAmount);

-- Clustered columnstore (data warehouse)
CREATE CLUSTERED COLUMNSTORE INDEX ix_Sales_Columnstore
ON Sales;
```

#### Filtered Index
- Index subset of rows
- Smaller index size
- Reduced maintenance

**Create:**
```sql
CREATE NONCLUSTERED INDEX ix_Orders_ActiveOrders
ON Orders(OrderDate)
WHERE Status = 'Active';
```

#### Unique Index
- Enforces uniqueness
- Can be clustered or nonclustered

**Create:**
```sql
CREATE UNIQUE NONCLUSTERED INDEX ix_Customers_Email
ON Customers(Email);
```

### Index Design Best Practices

#### Column Selection

**Index Key Columns:**
- Columns in WHERE clauses
- JOIN columns
- ORDER BY columns
- GROUP BY columns

**Include Columns:**
- SELECT list columns
- Avoid in key if not filtering/sorting
- Reduce key lookups

**Column Order:**
1. Equality predicates first
2. Inequality predicates next
3. Selectivity (most selective first)

**Example:**
```sql
-- Query: WHERE City = 'Seattle' AND StateID = 5 AND PostalCode > '98000'
-- Good index:
CREATE INDEX ix_Address_City_State_Postal
ON Address(City, StateID, PostalCode);

-- Better (if City is more selective):
CREATE INDEX ix_Address_State_City_Postal
ON Address(StateID, City, PostalCode);
```

#### Index Maintenance

**Fragmentation:**
```sql
-- Check index fragmentation
SELECT
    OBJECT_NAME(ips.object_id) AS table_name,
    i.name AS index_name,
    ips.index_type_desc,
    ips.avg_fragmentation_in_percent,
    ips.page_count
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'SAMPLED') ips
JOIN sys.indexes i ON ips.object_id = i.object_id AND ips.index_id = i.index_id
WHERE ips.avg_fragmentation_in_percent > 10
    AND ips.page_count > 1000
ORDER BY ips.avg_fragmentation_in_percent DESC;

-- Rebuild index (>30% fragmentation)
ALTER INDEX ix_Orders_OrderDate ON Orders REBUILD;

-- Reorganize index (10-30% fragmentation)
ALTER INDEX ix_Orders_OrderDate ON Orders REORGANIZE;

-- Rebuild all indexes on table
ALTER INDEX ALL ON Orders REBUILD;
```

**Update Statistics:**
```sql
-- Update statistics for table
UPDATE STATISTICS Orders;

-- Update specific statistics
UPDATE STATISTICS Orders ix_Orders_OrderDate;

-- Update all statistics in database
EXEC sp_updatestats;

-- Auto-update statistics (default enabled)
ALTER DATABASE MyDB SET AUTO_UPDATE_STATISTICS ON;
```

### Index Tuning Advisor

**Database Engine Tuning Advisor (DTA):**
1. Capture workload (SQL Profiler/Extended Events)
2. Run DTA against workload
3. Review recommendations
4. Implement suggested indexes

**Azure SQL Database Advisor:**
- Automatic index recommendations
- Create index recommendations
- Drop index recommendations
- Available in Azure portal

**Implement Recommendation:**
```sql
-- Azure portal provides script
CREATE NONCLUSTERED INDEX [ix_recommended_name]
ON [schema].[table] ([column1], [column2])
INCLUDE ([column3]);
```

---

## Query Tuning Techniques

### Parameterization

#### Benefits
- Plan reuse
- Reduced compilation
- Protection from SQL injection
- Consistent performance

**Example:**
```sql
-- Bad (ad-hoc)
SELECT * FROM Orders WHERE CustomerID = 123;
SELECT * FROM Orders WHERE CustomerID = 456;
-- Creates separate plans

-- Good (parameterized)
DECLARE @CustomerID INT = 123;
SELECT * FROM Orders WHERE CustomerID = @CustomerID;
-- Reuses plan

-- Stored procedure (best)
CREATE PROCEDURE GetOrdersByCustomer
    @CustomerID INT
AS
BEGIN
    SELECT * FROM Orders WHERE CustomerID = @CustomerID;
END;
```

#### Parameter Sniffing

**Problem:** Plan optimized for first parameter value.

**Solutions:**
```sql
-- Option 1: RECOMPILE hint
SELECT * FROM Orders WHERE CustomerID = @CustomerID
OPTION (RECOMPILE);

-- Option 2: OPTIMIZE FOR hint
SELECT * FROM Orders WHERE CustomerID = @CustomerID
OPTION (OPTIMIZE FOR (@CustomerID = 100));

-- Option 3: OPTIMIZE FOR UNKNOWN
SELECT * FROM Orders WHERE CustomerID = @CustomerID
OPTION (OPTIMIZE FOR (@CustomerID UNKNOWN));

-- Option 4: Local variable
DECLARE @LocalCustomerID INT = @CustomerID;
SELECT * FROM Orders WHERE CustomerID = @LocalCustomerID;
```

### Avoid Common Performance Issues

#### SELECT *
**Problem:** Returns unnecessary columns, prevents covering indexes.

```sql
-- Bad
SELECT * FROM Orders WHERE OrderDate > '2023-01-01';

-- Good
SELECT OrderID, CustomerID, OrderDate, TotalAmount
FROM Orders
WHERE OrderDate > '2023-01-01';
```

#### Implicit Conversions
**Problem:** Prevents index usage, adds CPU overhead.

```sql
-- Bad (VarcharColumn is VARCHAR, comparing to INT)
SELECT * FROM MyTable WHERE VarcharColumn = 123;

-- Good
SELECT * FROM MyTable WHERE VarcharColumn = '123';

-- Detect implicit conversions in execution plan
-- Look for CONVERT_IMPLICIT warnings
```

#### Functions on Indexed Columns
**Problem:** Prevents index seek.

```sql
-- Bad
SELECT * FROM Orders
WHERE YEAR(OrderDate) = 2023;

-- Good
SELECT * FROM Orders
WHERE OrderDate >= '2023-01-01' AND OrderDate < '2024-01-01';

-- Bad
SELECT * FROM Customers
WHERE UPPER(LastName) = 'SMITH';

-- Good (if case-insensitive collation)
SELECT * FROM Customers WHERE LastName = 'Smith';

-- Or create computed column with index
ALTER TABLE Customers ADD LastNameUpper AS UPPER(LastName);
CREATE INDEX ix_Customers_LastNameUpper ON Customers(LastNameUpper);
```

#### OR Conditions
**Problem:** May not use indexes efficiently.

```sql
-- May be inefficient
SELECT * FROM Orders
WHERE CustomerID = 123 OR OrderDate = '2023-01-01';

-- Consider UNION ALL
SELECT * FROM Orders WHERE CustomerID = 123
UNION ALL
SELECT * FROM Orders WHERE OrderDate = '2023-01-01' AND CustomerID <> 123;
```

#### Correlated Subqueries
**Problem:** Executes subquery for each row.

```sql
-- Bad
SELECT CustomerID, (SELECT MAX(OrderDate) FROM Orders o WHERE o.CustomerID = c.CustomerID) AS LastOrderDate
FROM Customers c;

-- Good (use JOIN)
SELECT c.CustomerID, MAX(o.OrderDate) AS LastOrderDate
FROM Customers c
LEFT JOIN Orders o ON c.CustomerID = o.CustomerID
GROUP BY c.CustomerID;

-- Or use APPLY
SELECT c.CustomerID, oa.LastOrderDate
FROM Customers c
OUTER APPLY (
    SELECT MAX(OrderDate) AS LastOrderDate
    FROM Orders o
    WHERE o.CustomerID = c.CustomerID
) oa;
```

### Query Hints

**Common Hints:**
```sql
-- Force index usage
SELECT * FROM Orders WITH (INDEX(ix_Orders_OrderDate))
WHERE OrderDate > '2023-01-01';

-- Force table scan
SELECT * FROM Orders WITH (INDEX(0))
WHERE OrderDate > '2023-01-01';

-- No lock (dirty read)
SELECT * FROM Orders WITH (NOLOCK)
WHERE OrderDate > '2023-01-01';

-- Read committed snapshot
SELECT * FROM Orders WITH (READCOMMITTEDLOCK)
WHERE OrderDate > '2023-01-01';

-- Force order of joins
SELECT * FROM Orders o
INNER JOIN Customers c ON o.CustomerID = c.CustomerID
OPTION (FORCE ORDER);

-- Maximum degree of parallelism
SELECT * FROM Orders
WHERE OrderDate > '2023-01-01'
OPTION (MAXDOP 4);

-- Recompile query
SELECT * FROM Orders WHERE CustomerID = @CustomerID
OPTION (RECOMPILE);
```

### Batch Operations

**Reduce Overhead:**
```sql
-- Bad (row-by-row)
WHILE EXISTS (SELECT * FROM #TempOrders)
BEGIN
    INSERT INTO Orders SELECT TOP 1 * FROM #TempOrders;
    DELETE TOP 1 FROM #TempOrders;
END;

-- Good (set-based)
INSERT INTO Orders SELECT * FROM #TempOrders;
```

**Chunking Large Operations:**
```sql
-- Update in batches to reduce locking
WHILE (1=1)
BEGIN
    UPDATE TOP (10000) Orders
    SET Status = 'Processed'
    WHERE Status = 'Pending';

    IF @@ROWCOUNT < 10000 BREAK;

    WAITFOR DELAY '00:00:01'; -- Breath between batches
END;
```

---

## Resource Monitoring and Management

### Resource Limits

#### SQL Database Limits (vCore Example)

**General Purpose (8 vCores):**
- Max workers: 800
- Max sessions: 30,000
- Max log rate: 36 MB/s
- Max data IOPS: 2,880
- Max tempdb size: 248 GB

**Business Critical (8 vCores):**
- Max workers: 800
- Max sessions: 30,000
- Max log rate: 48 MB/s
- Max data IOPS: 18,000 (local SSD)
- Max tempdb size: 248 GB

**Check Current Usage:**
```sql
-- Resource governance in Azure SQL
SELECT * FROM sys.dm_user_db_resource_governance;

-- Current DTU/vCore usage
SELECT
    end_time,
    avg_cpu_percent,
    avg_data_io_percent,
    avg_log_write_percent,
    avg_memory_usage_percent
FROM sys.dm_db_resource_stats
ORDER BY end_time DESC;
```

### Memory Management

**Memory Grants:**
```sql
-- Queries waiting for memory
SELECT
    session_id,
    wait_time,
    wait_type,
    granted_memory_kb,
    required_memory_kb,
    text
FROM sys.dm_exec_query_memory_grants qmg
CROSS APPLY sys.dm_exec_sql_text(qmg.sql_handle);

-- Queries with excessive memory grants
SELECT
    r.session_id,
    r.granted_query_memory * 8 AS granted_memory_kb,
    qt.text,
    qp.query_plan
FROM sys.dm_exec_requests r
CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) qt
CROSS APPLY sys.dm_exec_query_plan(r.plan_handle) qp
WHERE r.granted_query_memory > 10000 -- More than 80MB
ORDER BY r.granted_query_memory DESC;
```

**Buffer Pool:**
```sql
-- Buffer pool usage by database
SELECT
    DB_NAME(database_id) AS database_name,
    COUNT(*) * 8 / 1024 AS buffer_pool_mb
FROM sys.dm_os_buffer_descriptors
GROUP BY database_id
ORDER BY buffer_pool_mb DESC;

-- Buffer pool usage by object
SELECT
    OBJECT_NAME(p.object_id) AS object_name,
    i.name AS index_name,
    COUNT(*) * 8 / 1024 AS buffer_pool_mb
FROM sys.dm_os_buffer_descriptors bd
JOIN sys.allocation_units au ON bd.allocation_unit_id = au.allocation_unit_id
JOIN sys.partitions p ON au.container_id = p.hobt_id
JOIN sys.indexes i ON p.object_id = i.object_id AND p.index_id = i.index_id
WHERE bd.database_id = DB_ID()
GROUP BY p.object_id, i.name
ORDER BY buffer_pool_mb DESC;
```

### Tempdb Usage

```sql
-- Tempdb space usage
SELECT
    SUM(user_object_reserved_page_count) * 8 / 1024 AS user_objects_mb,
    SUM(internal_object_reserved_page_count) * 8 / 1024 AS internal_objects_mb,
    SUM(version_store_reserved_page_count) * 8 / 1024 AS version_store_mb,
    SUM(unallocated_extent_page_count) * 8 / 1024 AS free_space_mb
FROM sys.dm_db_file_space_usage;

-- Session tempdb usage
SELECT
    r.session_id,
    r.request_id,
    tsu.user_objects_alloc_page_count * 8 / 1024 AS user_objects_mb,
    tsu.internal_objects_alloc_page_count * 8 / 1024 AS internal_objects_mb,
    qt.text
FROM sys.dm_db_task_space_usage tsu
JOIN sys.dm_exec_requests r ON tsu.session_id = r.session_id AND tsu.request_id = r.request_id
CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) qt
WHERE tsu.user_objects_alloc_page_count > 0 OR tsu.internal_objects_alloc_page_count > 0
ORDER BY tsu.internal_objects_alloc_page_count DESC;
```

### Blocking and Deadlocks

**Current Blocking:**
```sql
-- Blocking chains
SELECT
    blocking.session_id AS blocking_session,
    blocked.session_id AS blocked_session,
    blocking_text.text AS blocking_query,
    blocked_text.text AS blocked_query,
    blocked.wait_type,
    blocked.wait_time / 1000 AS wait_time_sec
FROM sys.dm_exec_requests blocked
LEFT JOIN sys.dm_exec_requests blocking ON blocked.blocking_session_id = blocking.session_id
CROSS APPLY sys.dm_exec_sql_text(blocking.sql_handle) blocking_text
CROSS APPLY sys.dm_exec_sql_text(blocked.sql_handle) blocked_text
WHERE blocked.blocking_session_id > 0;

-- Kill blocking session (use carefully)
KILL 123; -- session_id
```

**Deadlock Detection:**
```sql
-- Enable deadlock trace flag (SQL on VM)
DBCC TRACEON(1222, -1);

-- Extended Events for deadlocks (Azure SQL Database)
CREATE EVENT SESSION [Deadlock_Tracking] ON DATABASE
ADD EVENT sqlserver.xml_deadlock_report
ADD TARGET package0.ring_buffer
WITH (MAX_MEMORY=4096 KB, MAX_DISPATCH_LATENCY=5 SECONDS);

ALTER EVENT SESSION [Deadlock_Tracking] ON DATABASE STATE = START;

-- View deadlocks from ring buffer
SELECT
    CAST(target_data AS XML) AS deadlock_xml
FROM sys.dm_xe_database_session_targets t
JOIN sys.dm_xe_database_sessions s ON t.event_session_address = s.address
WHERE s.name = 'Deadlock_Tracking';
```

---

## Troubleshooting Performance Issues

### Performance Troubleshooting Methodology

**Step 1: Identify Symptoms**
- Slow queries
- Timeouts
- High CPU
- High IO
- Blocking

**Step 2: Gather Information**
- Metrics (CPU, IO, DTU)
- Query Store data
- DMV queries
- Execution plans
- Wait statistics

**Step 3: Analyze Root Cause**
- Missing indexes
- Query plan issues
- Resource bottlenecks
- Blocking/deadlocks
- Configuration issues

**Step 4: Implement Solution**
- Add/modify indexes
- Rewrite queries
- Scale resources
- Tune configuration
- Implement caching

**Step 5: Validate and Monitor**
- Test changes
- Monitor metrics
- Compare performance
- Continuous monitoring

### Common Performance Issues and Solutions

#### High CPU Usage

**Causes:**
- Inefficient queries
- Missing indexes
- Parameter sniffing
- Excessive recompilation

**Investigation:**
```sql
-- Find CPU-intensive queries
SELECT TOP 20
    qs.total_worker_time / qs.execution_count / 1000.0 AS avg_cpu_ms,
    qs.total_worker_time / 1000.0 AS total_cpu_ms,
    qs.execution_count,
    qt.text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) qt
ORDER BY qs.total_worker_time DESC;
```

**Solutions:**
- Optimize queries
- Add missing indexes
- Use query hints
- Update statistics
- Consider scaling up

#### High IO

**Causes:**
- Table scans
- Missing indexes
- Large result sets
- Excessive sorting

**Investigation:**
```sql
-- Find IO-intensive queries
SELECT TOP 20
    qs.total_logical_reads / qs.execution_count AS avg_logical_reads,
    qs.total_logical_reads,
    qs.execution_count,
    qt.text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) qt
ORDER BY qs.total_logical_reads DESC;
```

**Solutions:**
- Create covering indexes
- Reduce data retrieved
- Partition large tables
- Use columnstore for analytics

#### Slow Query Performance

**Investigation Steps:**
1. Get execution plan
2. Check for table scans
3. Look for missing indexes
4. Check statistics
5. Review wait statistics
6. Check for parameter sniffing

**Tools:**
- Query Store
- Execution plans
- DMVs
- Azure SQL Insights

---

## Automatic Tuning

### Automatic Tuning Features (Azure SQL)

#### Force Last Good Plan
Automatically detects and fixes query plan regression.

**How it works:**
1. Detects performance regression
2. Compares current plan with previous plans
3. Forces last good plan
4. Monitors for improvement
5. Reverts if no improvement

**Enable:**
```sql
ALTER DATABASE [MyDB]
SET AUTOMATIC_TUNING (FORCE_LAST_GOOD_PLAN = ON);

-- Check status
SELECT name, desired_state_desc, actual_state_desc, reason_desc
FROM sys.database_automatic_tuning_options;
```

#### Create Index
Automatically creates recommended indexes.

**Enable:**
```sql
ALTER DATABASE [MyDB]
SET AUTOMATIC_TUNING (CREATE_INDEX = ON);
```

#### Drop Index
Automatically drops unused indexes.

**Enable:**
```sql
ALTER DATABASE [MyDB]
SET AUTOMATIC_TUNING (DROP_INDEX = ON);
```

### Automatic Tuning Recommendations

**View Recommendations:**
```sql
SELECT
    reason,
    score,
    state_transition_reason,
    CAST(state AS nvarchar(50)) AS state,
    CAST(details AS nvarchar(max)) AS details
FROM sys.dm_db_tuning_recommendations;
```

**Azure Portal:**
- Performance Recommendations
- Automatic tuning settings
- Recommendation history
- Impact analysis

### Best Practices

1. **Enable Query Store** - Required for automatic tuning
2. **Monitor Recommendations** - Review before auto-applying
3. **Test in Non-Prod** - Validate recommendations
4. **Set Baselines** - Know normal performance
5. **Regular Review** - Check tuning actions
6. **Combine with Manual** - Auto-tuning supplements, not replaces

---

## Exam Tips

### Key Concepts

**Monitoring Tools:**
- Azure Monitor: Metrics, logs, alerts
- Query Store: Query performance history
- DMVs: Real-time performance data
- Intelligent Insights: AI-powered diagnostics

**Performance Metrics:**
- CPU, IO, Memory percentage
- DTU/vCore usage
- Waits and latency
- Query execution statistics

**Index Types:**
- Clustered: Physical order, one per table
- Nonclustered: Separate structure, multiple allowed
- Columnstore: Columnar storage, analytics
- Filtered: Index subset of rows

**Query Tuning:**
- Use execution plans
- Avoid SELECT *
- Avoid functions on indexed columns
- Parameterize queries
- Use covering indexes

**Automatic Tuning:**
- Force last good plan
- Create index
- Drop index (unused)
- Requires Query Store

### Common Scenarios

**Scenario: Slow Query Performance**
1. Get execution plan
2. Check for table scans
3. Review missing index DMV
4. Add covering index
5. Update statistics

**Scenario: High CPU Usage**
1. Find CPU-intensive queries (DMV)
2. Review execution plans
3. Optimize queries
4. Add missing indexes
5. Consider scaling

**Scenario: Query Regression**
1. Enable Query Store
2. Compare plan performance
3. Force last good plan
4. Enable automatic tuning
5. Monitor improvement

**Scenario: Blocking Issues**
1. Identify blocking sessions
2. Review lock modes
3. Optimize long transactions
4. Consider isolation levels
5. Implement retry logic

### Important DMVs

- **sys.dm_exec_query_stats** - Query execution statistics
- **sys.dm_db_index_usage_stats** - Index usage
- **sys.dm_db_missing_index_details** - Missing indexes
- **sys.dm_os_wait_stats** - Wait statistics
- **sys.dm_exec_requests** - Current requests
- **sys.dm_db_resource_stats** - Resource usage (Azure SQL)

### Quick Reference

**Enable Query Store:**
```sql
ALTER DATABASE MyDB SET QUERY_STORE = ON;
```

**Force Execution Plan:**
```sql
EXEC sp_query_store_force_plan @query_id = X, @plan_id = Y;
```

**Rebuild Index:**
```sql
ALTER INDEX ix_name ON table REBUILD;
```

**Update Statistics:**
```sql
UPDATE STATISTICS table;
```

**Enable Automatic Tuning:**
```sql
ALTER DATABASE MyDB SET AUTOMATIC_TUNING (FORCE_LAST_GOOD_PLAN = ON);
```

### Study Focus

1. **Master Query Store** - Key tool for performance monitoring
2. **Understand Execution Plans** - Critical for query tuning
3. **Know Index Types** - When to use each
4. **Learn Key DMVs** - Essential for troubleshooting
5. **Practice Tuning Scenarios** - Hands-on experience
6. **Understand Automatic Tuning** - Azure SQL capability

### Final Checklist

- [ ] Query Store configuration and usage
- [ ] Reading and analyzing execution plans
- [ ] Index design and optimization
- [ ] Key performance DMVs
- [ ] Wait statistics analysis
- [ ] Automatic tuning features
- [ ] Resource monitoring and limits
- [ ] Blocking and deadlock resolution
- [ ] Query optimization techniques
- [ ] Performance troubleshooting methodology
