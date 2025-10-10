# Analytics and Reporting in Microsoft Fabric

## Table of Contents
1. [Power BI Integration](#power-bi-integration)
2. [Data Warehouse in Fabric](#data-warehouse-in-fabric)
3. [Real-Time Analytics](#real-time-analytics)
4. [Semantic Models (Datasets)](#semantic-models-datasets)
5. [DirectLake Mode](#directlake-mode)
6. [Performance Optimization](#performance-optimization)
7. [Exam Tips](#exam-tips)

---

## Power BI Integration

### Unified Analytics Platform

**Fabric + Power BI:**
- Native integration
- Single workspace
- Shared security model
- OneLake storage
- Direct Lake connectivity

**Benefits:**
- No data duplication
- Faster refresh
- Real-time insights
- Simplified architecture
- Reduced costs

### Power BI Items in Fabric

#### Reports
- Interactive visualizations
- Connected to semantic models
- Published to workspace
- Mobile-optimized views
- Export to PDF, PowerPoint

**Creation:**
- Power BI Desktop
- Power BI Service
- Direct from semantic model

#### Dashboards
- Pinned visuals from reports
- Tiles from multiple reports
- Real-time data
- Q&A natural language
- Mobile layouts

#### Semantic Models
- Data model (formerly dataset)
- Relationships and measures
- Row-level security
- Refresh schedules
- Query caching

### Data Connectivity

**Import Mode:**
- Data cached in Power BI
- Fast query performance
- Scheduled refresh required
- Storage limits apply

**DirectQuery:**
- Query source directly
- Real-time data
- No data refresh needed
- Source performance critical

**Direct Lake (Fabric):**
- Read Delta tables directly
- No import or DirectQuery
- Best of both worlds
- Automatic refresh
- Fabric workspaces only

**Composite Models:**
- Mix Import and DirectQuery
- Dual storage mode
- Aggregations
- Optimal performance

---

## Data Warehouse in Fabric

### Architecture

**Characteristics:**
- Fully managed SQL engine
- Separated compute and storage
- OneLake integration
- T-SQL interface
- Power BI optimized

**Components:**
1. Tables and views
2. Stored procedures
3. Functions
4. SQL analytics endpoint
5. Automatic indexing

### Creating Warehouse

```sql
-- Warehouse created via UI or API
-- Tables created with T-SQL

CREATE TABLE Sales (
    SaleID INT NOT NULL,
    OrderDate DATE NOT NULL,
    CustomerID INT NOT NULL,
    ProductID INT NOT NULL,
    Quantity INT NOT NULL,
    Amount DECIMAL(10,2) NOT NULL
);
```

### Data Loading

#### Using Pipelines
```json
{
    "name": "LoadWarehouse",
    "activities": [
        {
            "name": "CopyToWarehouse",
            "type": "Copy",
            "source": { "type": "LakehouseTable" },
            "sink": {
                "type": "WarehouseTable",
                "preCopyScript": "TRUNCATE TABLE Sales"
            }
        }
    ]
}
```

#### Using T-SQL (COPY INTO)
```sql
COPY INTO Sales
FROM 'https://account.dfs.core.windows.net/container/sales/*.parquet'
WITH (
    FILE_TYPE = 'PARQUET',
    CREDENTIAL = (IDENTITY = 'Managed Identity')
);
```

#### Using Dataflows
- Dataflow Gen2 destination
- Power Query transformations
- Scheduled refresh
- Incremental loads

### Table Types

#### Fact Tables
```sql
CREATE TABLE FactSales (
    SaleKey INT NOT NULL,
    DateKey INT NOT NULL,
    CustomerKey INT NOT NULL,
    ProductKey INT NOT NULL,
    Quantity INT,
    Amount DECIMAL(10,2),
    Cost DECIMAL(10,2)
);
```

#### Dimension Tables
```sql
CREATE TABLE DimCustomer (
    CustomerKey INT NOT NULL,
    CustomerID INT NOT NULL,
    CustomerName VARCHAR(100),
    City VARCHAR(50),
    State VARCHAR(50),
    Country VARCHAR(50)
);
```

#### Slowly Changing Dimensions (SCD)

**Type 1 (Overwrite):**
```sql
UPDATE DimCustomer
SET City = 'New York', State = 'NY'
WHERE CustomerID = 123;
```

**Type 2 (History):**
```sql
CREATE TABLE DimCustomer (
    CustomerKey INT IDENTITY(1,1) NOT NULL,
    CustomerID INT NOT NULL,
    CustomerName VARCHAR(100),
    City VARCHAR(50),
    State VARCHAR(50),
    EffectiveDate DATE NOT NULL,
    ExpirationDate DATE,
    IsCurrent BIT NOT NULL,
    PRIMARY KEY (CustomerKey)
);

-- Insert new version
INSERT INTO DimCustomer (CustomerID, CustomerName, City, State, EffectiveDate, IsCurrent)
VALUES (123, 'John Doe', 'Seattle', 'WA', '2024-01-15', 1);

-- Update old version
UPDATE DimCustomer
SET ExpirationDate = '2024-01-14', IsCurrent = 0
WHERE CustomerID = 123 AND IsCurrent = 1;
```

### Views and Materialized Views

```sql
-- Regular view
CREATE VIEW vwSalesSummary AS
SELECT
    c.CustomerName,
    p.ProductName,
    SUM(s.Quantity) AS TotalQuantity,
    SUM(s.Amount) AS TotalAmount
FROM FactSales s
JOIN DimCustomer c ON s.CustomerKey = c.CustomerKey
JOIN DimProduct p ON s.ProductKey = p.ProductKey
GROUP BY c.CustomerName, p.ProductName;

-- Fabric auto-creates materialized views for performance
```

### Security

#### Row-Level Security (RLS)
```sql
-- Create security predicate function
CREATE FUNCTION dbo.fn_securitypredicate(@Region AS VARCHAR(50))
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN SELECT 1 AS fn_securitypredicate_result
WHERE @Region = USER_NAME() OR USER_NAME() = 'Admin';

-- Create security policy
CREATE SECURITY POLICY RegionSecurityPolicy
ADD FILTER PREDICATE dbo.fn_securitypredicate(Region)
ON dbo.Sales
WITH (STATE = ON);
```

#### Column-Level Security
```sql
GRANT SELECT ON Sales (SaleID, OrderDate, Amount) TO DataAnalyst;
REVOKE SELECT ON Sales (Cost, Margin) TO DataAnalyst;
```

---

## Real-Time Analytics

### Eventhouse and KQL Database

**Eventhouse:**
- Real-time event processing
- KQL (Kusto Query Language)
- High-throughput ingestion
- Time-series optimized
- Streaming analytics

**KQL Database:**
- Part of Eventhouse
- Columnar storage
- Automatic indexing
- Retention policies
- Queried with KQL

### Data Ingestion

#### Event Streams
```json
{
    "name": "IoTStream",
    "source": {
        "type": "EventHub",
        "connectionString": "...",
        "consumerGroup": "$Default"
    },
    "destination": {
        "type": "KQLDatabase",
        "table": "IoTEvents"
    }
}
```

#### Streaming Dataflow
- Real-time transformations
- Event-by-event processing
- Windowing operations
- Multiple destinations

### KQL Queries

```kql
// Basic query
Events
| where Timestamp > ago(1h)
| where EventType == "Error"
| project Timestamp, Message, Source

// Aggregation
Events
| summarize Count=count(), AvgValue=avg(Value) by bin(Timestamp, 5m), DeviceId

// Time series
Events
| make-series AvgTemp=avg(Temperature) default=0 on Timestamp step 1h by DeviceId

// Joins
Events
| join kind=inner (Devices) on DeviceId
| project Timestamp, DeviceId, DeviceName, Temperature

// Window functions
Events
| extend PrevValue = prev(Value, 1)
| where Value > PrevValue * 1.5  // Spike detection
```

### Real-Time Dashboards

**Features:**
- Auto-refresh
- KQL-based visualizations
- Time range selection
- Drill-down capabilities
- Parameter filters

**Creation:**
1. Create KQL query
2. Add to dashboard
3. Configure refresh interval
4. Set parameters
5. Customize visualization

---

## Semantic Models (Datasets)

### Data Model Design

#### Star Schema
**Fact Table (Center):**
- Measures (numeric values)
- Foreign keys to dimensions
- Grain definition

**Dimension Tables (Points):**
- Descriptive attributes
- Hierarchies
- Primary keys

```dax
// Fact table
FactSales[SaleKey]
FactSales[DateKey]
FactSales[CustomerKey]
FactSales[Quantity]
FactSales[Amount]

// Dimension tables
DimDate[DateKey]
DimDate[Date]
DimDate[Year]
DimDate[Month]
DimDate[Quarter]

DimCustomer[CustomerKey]
DimCustomer[CustomerName]
DimCustomer[City]
DimCustomer[State]
```

#### Relationships

**One-to-Many (Standard):**
```
DimCustomer[CustomerKey] --1--* FactSales[CustomerKey]
```

**Configuration:**
- Cardinality: One-to-many
- Cross-filter direction: Single or Both
- Active/Inactive relationships

**Many-to-Many:**
```
FactSales -* Bridge *- DimPromotion
```

### DAX Measures

#### Basic Measures
```dax
Total Sales = SUM(FactSales[Amount])

Total Quantity = SUM(FactSales[Quantity])

Average Sale = AVERAGE(FactSales[Amount])

Count Orders = COUNTROWS(FactSales)
```

#### Time Intelligence
```dax
Sales YTD = TOTALYTD([Total Sales], DimDate[Date])

Sales Previous Year = CALCULATE([Total Sales], SAMEPERIODLASTYEAR(DimDate[Date]))

Sales YoY Growth =
DIVIDE(
    [Total Sales] - [Sales Previous Year],
    [Sales Previous Year]
)

Sales MTD = TOTALMTD([Total Sales], DimDate[Date])

Sales QTD = TOTALQTD([Total Sales], DimDate[Date])
```

#### Context Manipulation
```dax
// CALCULATE - Modify filter context
Sales in USA = CALCULATE([Total Sales], DimCustomer[Country] = "USA")

// ALL - Remove filters
All Customers Sales = CALCULATE([Total Sales], ALL(DimCustomer))

// FILTER - Apply custom filter
High Value Sales =
CALCULATE(
    [Total Sales],
    FILTER(FactSales, FactSales[Amount] > 1000)
)

// RELATED - Get related table column
Customer City = RELATED(DimCustomer[City])
```

#### Advanced Measures
```dax
// Running total
Running Total =
CALCULATE(
    [Total Sales],
    FILTER(
        ALL(DimDate[Date]),
        DimDate[Date] <= MAX(DimDate[Date])
    )
)

// Rank
Customer Rank =
RANKX(
    ALL(DimCustomer[CustomerName]),
    [Total Sales],
    ,
    DESC,
    Dense
)

// Pareto (80/20)
Cumulative % =
VAR CurrentSales = [Total Sales]
VAR TotalSales = CALCULATE([Total Sales], ALL(DimProduct))
VAR CumulativeSales =
    CALCULATE(
        [Total Sales],
        FILTER(
            ALL(DimProduct),
            [Total Sales] >= CurrentSales
        )
    )
RETURN
    DIVIDE(CumulativeSales, TotalSales)
```

### Calculated Columns vs Measures

**Calculated Columns:**
```dax
// Stored in table, calculated during refresh
Profit = FactSales[Amount] - FactSales[Cost]

Full Name = DimCustomer[FirstName] & " " & DimCustomer[LastName]
```

**When to Use:**
- Filtering/slicing
- Row-level calculations
- Grouping

**Measures:**
```dax
// Calculated at query time
Total Profit = SUM(FactSales[Profit])
```

**When to Use:**
- Aggregations
- Dynamic calculations
- Better performance for large models

### Hierarchies

```dax
// Geographic hierarchy
Geography
    Country
        State
            City

// Time hierarchy
Date
    Year
        Quarter
            Month
                Day
```

**Benefits:**
- Drill-down/up
- Natural grouping
- Better user experience

---

## DirectLake Mode

### Overview
New storage mode exclusive to Fabric.

**Characteristics:**
- Read Delta tables directly from OneLake
- No import or DirectQuery
- Automatic query optimization
- Fallback to DirectQuery if needed
- Lakehouse and Warehouse support

### How It Works

**Traditional:**
```
Delta Table → Import → Power BI Model → Report
           OR
Delta Table → DirectQuery → Report (every query hits source)
```

**Direct Lake:**
```
Delta Table ← Power BI Report (direct read, intelligent caching)
```

### Configuration

**Prerequisites:**
- Fabric capacity (F64+)
- Lakehouse or Warehouse
- Delta/Parquet tables
- Fabric workspace

**Creating Direct Lake Model:**
1. Create semantic model from Lakehouse/Warehouse
2. Select Direct Lake mode
3. Choose tables
4. Define relationships and measures
5. Publish to Power BI

**Limitations:**
- Calculated columns limited
- Complex DAX may trigger fallback
- Some DirectQuery limitations apply

### Fallback Behavior

**Triggers Fallback to DirectQuery:**
- Unsupported DAX functions
- Complex calculated columns
- Object-level security (OLS)
- V-Order optimization not applied

**Monitor Fallback:**
```dax
// Performance Analyzer in Power BI Desktop
// Check query execution mode
```

### Benefits

| Aspect | Import | DirectQuery | Direct Lake |
|--------|--------|-------------|-------------|
| Performance | Fast | Slow | Fast |
| Data Freshness | Scheduled | Real-time | Real-time |
| Data Size | Limited | Unlimited | Unlimited |
| Query Source | Cache | Source | OneLake |
| Refresh Needed | Yes | No | No |

---

## Performance Optimization

### Semantic Model Optimization

#### 1. Reduce Model Size
```dax
// Remove unused columns
// Use calculated measures instead of calculated columns
// Remove unnecessary tables

// Compress data
// Integer instead of strings for IDs
// Reduce cardinality of text columns
```

#### 2. Optimize Relationships
- Use integer keys
- Avoid many-to-many when possible
- Set correct cardinality
- Use single direction cross-filtering

#### 3. Aggregations
```dax
// Pre-aggregated tables for large facts
CREATE TABLE FactSalesAgg AS
SELECT
    DateKey,
    CustomerKey,
    SUM(Quantity) AS TotalQuantity,
    SUM(Amount) AS TotalAmount
FROM FactSales
GROUP BY DateKey, CustomerKey;

// Power BI automatically uses aggregation table
```

#### 4. Incremental Refresh

**Configure in Power BI:**
1. Define RangeStart and RangeEnd parameters
2. Filter table using parameters
3. Configure incremental refresh policy
   - Archive: Keep last X years
   - Refresh: Refresh last X days
4. Publish to service

**Benefits:**
- Faster refresh times
- Reduced resource usage
- Lower data transfer

### Query Performance

#### 1. Use Measures Over Calculated Columns
```dax
// Good (Measure)
Total Sales = SUM(FactSales[Amount])

// Avoid (Calculated Column for aggregation)
Total Sales Column = CALCULATE(SUM(FactSales[Amount]))
```

#### 2. Optimize DAX
```dax
// Bad - Multiple CALCULATE calls
Sales Metric =
    CALCULATE(SUM(FactSales[Amount])) +
    CALCULATE(SUM(FactSales[Shipping]))

// Good - Single CALCULATE
Sales Metric =
    CALCULATE(SUM(FactSales[Amount]) + SUM(FactSales[Shipping]))

// Bad - FILTER iterates all rows
High Value =
    CALCULATE(
        [Total Sales],
        FILTER(FactSales, FactSales[Amount] > 1000)
    )

// Good - Use KEEPFILTERS or simple filter
High Value =
    CALCULATE(
        [Total Sales],
        FactSales[Amount] > 1000
    )
```

#### 3. Variables for Performance
```dax
// Bad - Expression evaluated multiple times
Margin % =
    DIVIDE(
        [Total Sales] - [Total Cost],
        [Total Sales]
    )

// Good - Use variables
Margin % =
VAR Sales = [Total Sales]
VAR Cost = [Total Cost]
RETURN
    DIVIDE(Sales - Cost, Sales)
```

### Report Performance

#### 1. Reduce Visuals Per Page
- Limit to 5-7 visuals per page
- Use bookmarks for multiple views
- Consider separate report pages

#### 2. Optimize Visuals
- Avoid complex custom visuals
- Use built-in visuals when possible
- Limit data points per visual
- Use TOP N filters

#### 3. Use Report-Level Filters
- Filter at report level, not visual level
- Reduces number of queries
- Better caching

#### 4. Disable Auto-Sync for Slicers
- Prevents excessive queries
- User clicks Apply button
- Better control

### Warehouse Performance

#### 1. Table Distribution
```sql
-- Fabric auto-distributes, but understand concepts

-- Round-robin (default for small tables)
-- Hash distribution (large fact tables)
-- Replicated (small dimension tables)
```

#### 2. Indexing
- Automatic in Fabric Warehouse
- Clustered columnstore index (default)
- Automatic statistics

#### 3. Partitioning
```sql
-- Partition large fact tables by date
-- Improves query performance
-- Easier maintenance
```

#### 4. Materialized Views
- Auto-created by Fabric
- Improves query performance
- Transparent to queries

---

## Exam Tips

### Key Concepts

**Power BI in Fabric:**
- Native integration with Lakehouse, Warehouse, Eventstream
- Direct Lake mode for best performance
- Semantic models (datasets) central to analytics
- Real-time analytics with KQL

**Data Warehouse:**
- Fully managed SQL engine
- T-SQL for queries and data manipulation
- Star schema for dimensional modeling
- SCD Types 1 and 2 for history

**Direct Lake:**
- Fabric-exclusive storage mode
- Reads Delta tables directly
- No import or refresh needed
- Fallback to DirectQuery if needed

**Real-Time Analytics:**
- Eventhouse and KQL Database
- KQL for queries
- Event streaming ingestion
- Real-time dashboards

**DAX:**
- Measures for aggregations
- Calculated columns for row-level
- Time intelligence functions
- Context manipulation (CALCULATE, FILTER, ALL)

### Common Scenarios

**Building Analytics Solution:**
1. Ingest data to Lakehouse (Pipeline/Dataflow)
2. Transform with Spark or Dataflow
3. Create Warehouse for dimensional model
4. Build semantic model with Direct Lake
5. Create Power BI reports and dashboards

**Implementing SCD Type 2:**
1. Add EffectiveDate, ExpirationDate, IsCurrent columns
2. On change: Insert new row, update old row
3. Use IsCurrent for current state queries
4. Use date range for historical queries

**Optimizing Report Performance:**
1. Use aggregations for large fact tables
2. Implement incremental refresh
3. Optimize DAX measures with variables
4. Reduce visuals per page
5. Use Direct Lake mode

**Real-Time Dashboard:**
1. Create Eventstream from source
2. Ingest to KQL Database
3. Write KQL queries for aggregations
4. Create Real-Time Dashboard
5. Configure auto-refresh

### Decision Matrix

**Storage Mode:**
- Import: Fast, static, scheduled refresh
- DirectQuery: Real-time, slower, large data
- Direct Lake: Fast, real-time, Fabric only
- Composite: Mix for optimal performance

**Lakehouse vs Warehouse:**
- Lakehouse: Data engineering, Spark, semi-structured
- Warehouse: BI/analytics, SQL, structured
- Both: Use Lakehouse for raw/transformed, Warehouse for curated

**Calculated Column vs Measure:**
- Calculated Column: Filtering, grouping, row-level
- Measure: Aggregations, dynamic, better performance

### Quick Reference

**DAX Functions:**
- SUM, COUNT, AVERAGE, MIN, MAX: Aggregations
- CALCULATE: Modify filter context
- FILTER: Custom filter
- ALL, ALLEXCEPT: Remove filters
- RELATED, RELATEDTABLE: Cross-table
- TOTALYTD, SAMEPERIODLASTYEAR: Time intelligence

**KQL Functions:**
- where: Filter rows
- summarize: Aggregate
- make-series: Time series
- join: Combine tables
- bin: Time bucketing
- ago: Relative time

**Warehouse T-SQL:**
- CREATE TABLE: Define schema
- COPY INTO: Bulk load
- INSERT, UPDATE, DELETE: DML
- CREATE VIEW: Virtual table
- CREATE SECURITY POLICY: RLS

**Direct Lake:**
- Prerequisites: Fabric capacity F64+, Lakehouse/Warehouse
- Benefits: Fast, real-time, no refresh
- Fallback: Complex DAX, OLS
- Best for: Large semantic models on Fabric

### Study Focus

1. **Power BI integration** in Fabric ecosystem
2. **Direct Lake mode** capabilities and limitations
3. **DAX fundamentals** - measures, time intelligence
4. **Data Warehouse** design - star schema, SCD
5. **KQL basics** for real-time analytics
6. **Performance optimization** techniques
7. **Semantic model design** best practices

### Final Checklist

- [ ] Direct Lake vs Import vs DirectQuery
- [ ] DAX measure creation and optimization
- [ ] Star schema design principles
- [ ] SCD implementation (Type 1 and 2)
- [ ] KQL query syntax
- [ ] Real-time analytics architecture
- [ ] Semantic model relationships
- [ ] Incremental refresh configuration
- [ ] Aggregations for performance
- [ ] Row-level security implementation
- [ ] Report performance optimization
- [ ] Lakehouse vs Warehouse use cases
