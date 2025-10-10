# Data Engineering with Microsoft Fabric

## Table of Contents
1. [Data Factory in Fabric](#data-factory-in-fabric)
2. [Synapse Data Engineering](#synapse-data-engineering)
3. [Lakehouses](#lakehouses)
4. [Data Pipelines](#data-pipelines)
5. [Dataflows Gen2](#dataflows-gen2)
6. [Notebooks and Spark](#notebooks-and-spark)
7. [Exam Tips](#exam-tips)

---

## Data Factory in Fabric

### Overview
Integrated data integration service within Microsoft Fabric.

**Key Features:**
- Visual pipeline designer
- 200+ data connectors
- Dataflows Gen2
- Copy activities
- Data transformation
- Orchestration and scheduling

### Components

#### 1. Data Pipelines
- Visual designer for ETL/ELT
- Activity-based workflows
- Control flow and data flow
- Parameters and variables
- Triggers and scheduling

#### 2. Dataflows Gen2
- Power Query-based transformations
- Self-service data preparation
- Mashup engine
- Query folding
- Incremental refresh

#### 3. Connectors
**Categories:**
- File-based (CSV, JSON, Parquet, Excel)
- Databases (SQL Server, Oracle, MySQL, PostgreSQL)
- Cloud services (Azure, AWS, GCP)
- SaaS applications (Salesforce, Dynamics, SharePoint)
- Streaming (Event Hubs, Kafka)

---

## Synapse Data Engineering

### Spark Pools in Fabric
**Characteristics:**
- Serverless Apache Spark
- Autoscaling
- Auto-pause (5-minute default)
- Multiple runtime versions
- Library management

**Configuration:**
```python
# Spark session configuration
%%configure
{
    "conf": {
        "spark.executor.memory": "8g",
        "spark.executor.cores": 4,
        "spark.dynamicAllocation.enabled": "true"
    }
}
```

### Spark Job Definitions
Reusable Spark applications.

**Components:**
- Main definition file (.py, .jar, .R)
- Command-line arguments
- Reference files and libraries
- Spark configuration

**Creation:**
```python
# Python Spark job
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("MyJob").getOrCreate()

# Read data
df = spark.read.format("delta").load("Tables/MyTable")

# Transform
result = df.filter(df.status == "active").select("id", "name", "value")

# Write
result.write.format("delta").mode("overwrite").save("Tables/ProcessedData")
```

### Delta Lake Integration
**Features:**
- ACID transactions
- Time travel
- Schema evolution
- Unified batch and streaming
- Scalable metadata

**Delta Operations:**
```python
# Read Delta table
df = spark.read.format("delta").load("Tables/Sales")

# Write with merge schema
df.write.format("delta") \
    .mode("overwrite") \
    .option("mergeSchema", "true") \
    .save("Tables/Sales")

# Time travel
df = spark.read.format("delta") \
    .option("versionAsOf", 5) \
    .load("Tables/Sales")

# Optimize table
spark.sql("OPTIMIZE delta.`Tables/Sales`")

# Vacuum old files
spark.sql("VACUUM delta.`Tables/Sales` RETAIN 168 HOURS")
```

---

## Lakehouses

### Architecture

#### Components
1. **Files Section**
   - Unstructured/semi-structured data
   - OneLake storage
   - Folders and files
   - Multiple formats supported

2. **Tables Section**
   - Structured data
   - Delta Lake format
   - Parquet-based
   - SQL queryable

3. **SQL Analytics Endpoint**
   - Read-only SQL access
   - T-SQL queries
   - Power BI integration
   - Automatically created

#### OneLake Integration
- Single data lake for organization
- Hierarchical namespace
- ADLS Gen2 compatible
- Automatic data management
- Unified security model

### Creating and Managing Lakehouses

#### Create Lakehouse
```python
# Via API or UI
{
    "displayName": "SalesLakehouse",
    "description": "Sales data lakehouse"
}
```

#### Load Data to Lakehouse
```python
# Using notebooks
df = spark.read.format("csv") \
    .option("header", "true") \
    .load("Files/raw/sales.csv")

# Write as managed Delta table
df.write.format("delta") \
    .mode("overwrite") \
    .saveAsTable("Sales")

# Or write to Files
df.write.format("parquet") \
    .mode("overwrite") \
    .save("Files/processed/sales.parquet")
```

### Lakehouse vs Warehouse

| Aspect | Lakehouse | Warehouse |
|--------|-----------|-----------|
| Storage | OneLake (Delta/Parquet) | Dedicated SQL storage |
| Access | Spark SQL, T-SQL | T-SQL |
| Data Types | Structured, semi-structured | Structured |
| Use Case | Data engineering, science | BI, analytics |
| Format | Delta Lake | Proprietary |
| Write | Spark, Pipelines, Dataflows | SQL, Pipelines, Dataflows |

### Shortcuts
Link external data without copying.

**Types:**
1. **OneLake Shortcuts**
   - Link to other Fabric workspaces
   - Within same tenant
   - Real-time access

2. **External Shortcuts**
   - ADLS Gen2
   - S3 (AWS)
   - Dataverse

**Creation:**
```python
# Create shortcut via UI or API
{
    "name": "ExternalData",
    "path": "Files/external",
    "target": {
        "type": "AzureDataLakeStorage",
        "connectionString": "...",
        "path": "/container/path"
    }
}
```

**Benefits:**
- No data duplication
- Real-time access
- Reduced storage costs
- Single source of truth

---

## Data Pipelines

### Pipeline Components

#### Activities

**1. Copy Activity**
```json
{
    "name": "CopyData",
    "type": "Copy",
    "inputs": [{
        "referenceName": "SourceDataset",
        "type": "DatasetReference"
    }],
    "outputs": [{
        "referenceName": "SinkDataset",
        "type": "DatasetReference"
    }],
    "typeProperties": {
        "source": {
            "type": "DelimitedTextSource"
        },
        "sink": {
            "type": "LakehouseTableSink"
        }
    }
}
```

**2. Notebook Activity**
```json
{
    "name": "TransformData",
    "type": "SparkNotebook",
    "typeProperties": {
        "notebook": {
            "referenceName": "TransformationNotebook",
            "type": "NotebookReference"
        },
        "parameters": {
            "date": "2024-01-15"
        }
    }
}
```

**3. Dataflow Activity**
```json
{
    "name": "PrepareData",
    "type": "ExecuteDataFlow",
    "typeProperties": {
        "dataflow": {
            "referenceName": "SalesDataflow",
            "type": "DataFlowReference"
        }
    }
}
```

**4. Control Flow Activities**
- ForEach Loop
- If Condition
- Switch
- Until Loop
- Wait
- Set Variable
- Append Variable

### Parameters and Variables

#### Pipeline Parameters
```json
{
    "parameters": {
        "StartDate": {
            "type": "string",
            "defaultValue": "2024-01-01"
        },
        "EndDate": {
            "type": "string"
        }
    }
}
```

**Usage:**
```json
"typeProperties": {
    "query": "SELECT * FROM Sales WHERE OrderDate >= '@{pipeline().parameters.StartDate}' AND OrderDate <= '@{pipeline().parameters.EndDate}'"
}
```

#### Variables
```json
{
    "variables": {
        "RecordCount": {
            "type": "Integer",
            "defaultValue": 0
        },
        "ErrorMessage": {
            "type": "String"
        }
    }
}
```

**Set Variable:**
```json
{
    "name": "SetCount",
    "type": "SetVariable",
    "typeProperties": {
        "variableName": "RecordCount",
        "value": "@activity('CopyData').output.rowsCopied"
    }
}
```

### Triggers

#### Schedule Trigger
```json
{
    "name": "DailySchedule",
    "type": "ScheduleTrigger",
    "typeProperties": {
        "recurrence": {
            "frequency": "Day",
            "interval": 1,
            "startTime": "2024-01-01T06:00:00Z",
            "timeZone": "UTC"
        }
    }
}
```

#### Event-Based Trigger
```json
{
    "name": "OnFileArrival",
    "type": "BlobEventsTrigger",
    "typeProperties": {
        "blobPathBeginsWith": "/container/input/",
        "blobPathEndsWith": ".csv",
        "events": ["Microsoft.Storage.BlobCreated"]
    }
}
```

### Error Handling

#### Activity-Level
```json
{
    "name": "CopyWithRetry",
    "type": "Copy",
    "policy": {
        "timeout": "0.12:00:00",
        "retry": 3,
        "retryIntervalInSeconds": 30
    },
    "userProperties": [],
    "typeProperties": { ... }
}
```

#### Pipeline-Level
```json
{
    "activities": [
        {
            "name": "MainActivity",
            "type": "Copy",
            "dependsOn": [],
            "onSuccess": {
                "activities": [{"name": "SuccessLog"}]
            },
            "onFailure": {
                "activities": [{"name": "ErrorLog"}]
            }
        }
    ]
}
```

---

## Dataflows Gen2

### Power Query Transformations

#### Common Transformations

**1. Filter Rows**
```M
// Power Query M
= Table.SelectRows(Source, each [Status] = "Active")
```

**2. Select Columns**
```M
= Table.SelectColumns(Source, {"OrderID", "CustomerID", "OrderDate", "Total"})
```

**3. Add Column**
```M
= Table.AddColumn(Source, "FullName", each [FirstName] & " " & [LastName])
```

**4. Group By**
```M
= Table.Group(Source, {"CustomerID"}, {
    {"TotalOrders", each Table.RowCount(_), Int64.Type},
    {"TotalAmount", each List.Sum([Amount]), type number}
})
```

**5. Merge Queries (Join)**
```M
= Table.NestedJoin(
    Orders, {"CustomerID"},
    Customers, {"CustomerID"},
    "CustomerInfo",
    JoinKind.LeftOuter
)
```

**6. Append Queries (Union)**
```M
= Table.Combine({Query1, Query2, Query3})
```

### Query Folding

**Definition:** Push transformations to source system.

**Benefits:**
- Faster performance
- Reduced data movement
- Less memory usage
- Leverage source compute

**Foldable Operations:**
- Filter
- Select columns
- Join (native sources)
- Group by (aggregations)
- Sort

**Non-Foldable:**
- Custom columns with complex logic
- Some M functions
- Mixed source queries

**Check Folding:**
- Right-click step → "View Native Query"
- If grayed out, not folding

### Incremental Refresh

**Configuration:**
```M
// Define RangeStart and RangeEnd parameters
Source = ...,
FilteredRows = Table.SelectRows(Source, each
    [ModifiedDate] >= RangeStart and
    [ModifiedDate] < RangeEnd
)
```

**Settings:**
- Archive data: Keep last X years
- Refresh data: Refresh last X days
- Detect data changes (optional)

**Benefits:**
- Faster refresh
- Reduced data transfer
- Lower compute costs

### Dataflow Destinations

**Supported:**
- Lakehouse (Files and Tables)
- Data Warehouse
- Azure SQL Database
- Dataverse
- Cloud storage (ADLS, Blob)

**Configuration:**
```M
// Lakehouse table destination
{
    "destination": "Lakehouse",
    "workspace": "MyWorkspace",
    "lakehouse": "SalesLakehouse",
    "table": "Customers",
    "updateMethod": "Replace"
}
```

**Update Methods:**
- Replace
- Append
- Delete + Insert (staging)

---

## Notebooks and Spark

### Notebook Capabilities

#### Supported Languages
- PySpark (Python)
- Scala
- SparkSQL
- SparkR
- .NET Spark (C#, F#)

**Magic Commands:**
```python
%%pyspark
# Python code

%%sql
-- SQL code

%%configure
# Spark configuration

%%help
# Show available commands
```

### Data Operations

#### Reading Data

**From Lakehouse:**
```python
# Read Delta table
df = spark.read.format("delta").load("Tables/Sales")

# Read from Files
df = spark.read.format("csv") \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .load("Files/raw/data.csv")

# Read Parquet
df = spark.read.parquet("Files/processed/data.parquet")

# Read JSON
df = spark.read.json("Files/raw/data.json")
```

**From External:**
```python
# Azure Blob Storage
df = spark.read.format("csv") \
    .option("header", "true") \
    .load("abfss://container@account.dfs.core.windows.net/path/file.csv")

# Using mssparkutils
files = mssparkutils.fs.ls("Files/raw")
for file in files:
    print(file.path)
```

#### Writing Data

**To Lakehouse:**
```python
# Write Delta table (managed)
df.write.format("delta") \
    .mode("overwrite") \
    .option("overwriteSchema", "true") \
    .saveAsTable("ProcessedSales")

# Write to Files
df.write.format("parquet") \
    .mode("overwrite") \
    .save("Files/processed/sales.parquet")

# Partition by column
df.write.format("delta") \
    .partitionBy("Year", "Month") \
    .mode("overwrite") \
    .saveAsTable("SalesPartitioned")
```

### Transformations

```python
# Filter
filtered_df = df.filter(df.Status == "Active")

# Select columns
selected_df = df.select("OrderID", "CustomerID", "Total")

# Add column
from pyspark.sql.functions import col, concat, lit
df_with_col = df.withColumn("FullName", concat(col("FirstName"), lit(" "), col("LastName")))

# Aggregate
from pyspark.sql.functions import sum, avg, count
aggregated = df.groupBy("CustomerID").agg(
    count("OrderID").alias("OrderCount"),
    sum("Total").alias("TotalSpent"),
    avg("Total").alias("AverageOrder")
)

# Join
result = orders.join(customers, orders.CustomerID == customers.CustomerID, "inner")

# Window functions
from pyspark.sql.window import Window
from pyspark.sql.functions import row_number

windowSpec = Window.partitionBy("CustomerID").orderBy(col("OrderDate").desc())
ranked = df.withColumn("rank", row_number().over(windowSpec))
```

### Performance Optimization

#### Caching
```python
# Cache DataFrame
df.cache()
# or
df.persist()

# Unpersist
df.unpersist()
```

#### Broadcast Join
```python
from pyspark.sql.functions import broadcast

# Broadcast small table
result = large_df.join(broadcast(small_df), "key")
```

#### Repartitioning
```python
# Increase partitions
df_repartitioned = df.repartition(100)

# Repartition by column
df_repartitioned = df.repartition(50, "CustomerID")

# Coalesce (reduce partitions, no shuffle)
df_coalesced = df.coalesce(10)
```

#### Predicate Pushdown
```python
# Filter early
df = spark.read.parquet("Files/large_dataset.parquet") \
    .filter("OrderDate >= '2024-01-01'")  # Pushed to read
```

### mssparkutils

**File Operations:**
```python
# List files
files = mssparkutils.fs.ls("Files/raw")

# Copy
mssparkutils.fs.cp("Files/source/file.csv", "Files/dest/file.csv")

# Move
mssparkutils.fs.mv("Files/old/file.csv", "Files/new/file.csv")

# Delete
mssparkutils.fs.rm("Files/temp/file.csv")
```

**Credentials:**
```python
# Get secret from Key Vault (if integrated)
secret = mssparkutils.credentials.getSecret("KeyVaultName", "SecretName")
```

**Notebook Utilities:**
```python
# Exit notebook
mssparkutils.notebook.exit("Success")

# Run another notebook
result = mssparkutils.notebook.run("OtherNotebook", 60, {"param1": "value1"})
```

---

## Exam Tips

### Key Concepts

**Data Factory:**
- Copy Activity: Data movement
- Dataflow Gen2: Power Query transformations
- Pipeline: Orchestration and control flow
- Triggers: Schedule and event-based

**Lakehouse:**
- Files: Unstructured/semi-structured
- Tables: Delta Lake (structured)
- SQL Analytics Endpoint: Read-only SQL
- Shortcuts: Link external data

**Spark:**
- Delta Lake: ACID, time travel, schema evolution
- Notebooks: Multi-language (Python, SQL, Scala)
- Optimization: Cache, broadcast, repartition
- mssparkutils: File and notebook utilities

**Dataflows Gen2:**
- Power Query M transformations
- Query folding for performance
- Incremental refresh for large datasets
- Multiple destination types

### Common Scenarios

**ETL Pipeline:**
1. Copy Activity: Ingest raw data to Lakehouse Files
2. Notebook Activity: Transform with Spark, write to Tables
3. Dataflow Activity: Additional transformations
4. Schedule: Daily trigger at 2 AM

**Incremental Load:**
1. Dataflow Gen2 with RangeStart/RangeEnd parameters
2. Filter based on ModifiedDate
3. Append to existing table
4. Optimize table periodically

**Multi-Source Integration:**
1. Copy Activities: Ingest from SQL, API, Files
2. Dataflow: Merge and transform
3. Lakehouse: Centralized storage
4. Warehouse: Serve analytics

**Performance Optimization:**
1. Check query folding in Dataflows
2. Use broadcast joins for small dimensions
3. Partition large tables
4. Cache frequently used DataFrames

### Decision Matrix

**Dataflow Gen2 vs Notebooks:**
- Dataflow Gen2: GUI-based, Power Query, easier for non-coders
- Notebooks: Code-based, complex logic, Spark optimizations

**Lakehouse vs Warehouse:**
- Lakehouse: Data engineering, Spark access, Delta format
- Warehouse: BI/analytics, SQL-only, optimized for queries

**OneLake Shortcut vs Copy:**
- Shortcut: Real-time access, no duplication, external data
- Copy: Independent copy, transformation needed, internal

### Quick Reference

**Pipeline Activities:**
- Copy: Data movement
- Notebook: Spark execution
- Dataflow: Power Query transformation
- ForEach: Iterate over collection
- If Condition: Conditional logic

**Dataflow Transformations:**
- Filter Rows: WHERE clause
- Select Columns: Project columns
- Group By: Aggregation
- Merge: JOIN
- Append: UNION

**Spark Operations:**
- read/write: I/O operations
- filter/select: Transformations
- groupBy/agg: Aggregations
- join: Combine DataFrames
- cache/persist: Optimization

**Delta Lake Commands:**
- OPTIMIZE: Compaction
- VACUUM: Cleanup old files
- Time travel: Version/timestamp queries
- MERGE: Upsert operations

### Study Focus

1. **Understand pipeline components** and when to use each
2. **Master Dataflow Gen2** transformations and query folding
3. **Know Lakehouse architecture** - Files vs Tables
4. **Learn Spark basics** - read, write, transform
5. **Understand Delta Lake** features and benefits
6. **Practice building pipelines** end-to-end
7. **Know optimization techniques** for performance

### Final Checklist

- [ ] Pipeline activity types and usage
- [ ] Dataflow Gen2 transformations
- [ ] Query folding principles
- [ ] Lakehouse vs Warehouse differences
- [ ] Shortcuts vs copying data
- [ ] Spark DataFrame operations
- [ ] Delta Lake capabilities
- [ ] Notebook magic commands
- [ ] mssparkutils functions
- [ ] Performance optimization techniques
- [ ] Incremental refresh configuration
- [ ] Trigger types and scheduling
