# Microsoft Fabric Analytics Overview

## Overview
Microsoft Fabric is a unified analytics platform that brings together data integration, data engineering, data warehousing, data science, real-time analytics, and business intelligence.

## Fabric Components

### OneLake
- **Unified data lake** for entire organization
- **Hierarchical namespace**
- **Parquet and Delta Lake** format
- **Single copy** of data, multiple workloads

### Data Factory (Fabric)
- **Data integration** and orchestration
- **Dataflows Gen2:** Power Query at scale
- **Data pipelines:** Copy and transform data
- **Similar to Azure Data Factory**

### Synapse Data Engineering
- **Apache Spark** for data engineering
- **Notebooks:** Interactive development
- **Spark job definitions**
- **Lakehouse:** Data storage and compute

### Synapse Data Warehouse
- **Enterprise** data warehouse
- **T-SQL** queries
- **Auto-tune and optimize**
- **Separates** storage and compute

### Synapse Real-Time Analytics
- **Event streaming** and analysis
- **Kusto Query Language (KQL)**
- **Time-series data**
- **IoT and log analytics**

### Power BI
- **Business intelligence** and reporting
- **Direct Lake mode:** Query OneLake directly
- **Interactive dashboards**
- **Natural language queries** (Q&A)

### Data Science
- **Machine learning** workflows
- **Notebooks:** Python, R, Scala
- **MLflow** integration
- **AutoML** capabilities

## Lakehouse Architecture

### What is a Lakehouse?
- **Combines:** Data lake + Data warehouse
- **Delta Lake** format (ACID transactions)
- **Schema enforcement**
- **Time travel** (versioning)

### Files vs Tables
**Files:**
- **Unstructured data**
- **Any format:** CSV, JSON, Parquet
- **Flexible** schema-on-read

**Tables:**
- **Structured data**
- **Delta Lake** format
- **ACID** transactions
- **Query with T-SQL or Spark SQL**

## Semantic Models

### Creating Semantic Models
- **Data modeling layer** for Power BI
- **Define relationships** between tables
- **Create measures and calculations** (DAX)
- **Reusable** across reports

### Direct Lake Mode
- **Query OneLake** directly
- **No data import** needed
- **Best performance**
- **Automatic refresh**

## Data Integration

### Dataflows Gen2
- **Power Query** engine
- **Transform data** at scale
- **Output to Lakehouse, Warehouse, or KQL DB**
- **Incremental refresh**

### Data Pipelines
- **Copy** activity for data movement
- **Transform** with notebooks or Spark
- **Schedule** and orchestrate
- **Integration** with Azure services

## Real-Time Analytics

### Eventhouse and KQL Databases
- **Event streaming**
- **Time-series data**
- **Real-time dashboards**
- **IoT scenarios**

### Streaming Data
- **Event Streams:** Kafka-compatible
- **Real-time processing**
- **Integration** with Fabric components

## Security and Governance

### Workspace Roles
- **Admin:** Full control
- **Member:** Create and edit
- **Contributor:** Edit existing
- **Viewer:** Read-only access

### Data Access
- **OneLake data access roles**
- **Row-level security** (RLS) in semantic models
- **Object-level security** (OLS)

### Monitoring
- **Monitoring hub:** Track activities
- **Metrics:** Performance and usage
- **Alerts:** Proactive notifications

## Best Practices

### Lakehouse Design
1. **Medallion architecture:** Bronze → Silver → Gold
2. **Use Delta Lake** for tables
3. **Partition** large tables appropriately
4. **Optimize** file sizes (128MB-1GB)

### Semantic Models
1. **Star schema** design (fact + dimension tables)
2. **Use Direct Lake** when possible
3. **Create calculated** columns and measures in DAX
4. **Implement RLS** for security

### Performance
1. **Optimize Spark** cluster sizes
2. **Use V-Order** for faster reads
3. **Partition** and **Z-order** data
4. **Cache** frequently accessed data

## Study Tips

### Key Concepts
- OneLake as unified data lake
- Lakehouse vs Warehouse vs KQL DB
- Semantic models and Direct Lake mode
- Dataflows vs Data Pipelines
- Medallion architecture (Bronze/Silver/Gold)

### Common Scenarios
1. **Data engineering** → Lakehouse with Spark
2. **Data warehousing** → Synapse Data Warehouse
3. **Real-time analytics** → KQL Database
4. **BI and reporting** → Power BI with semantic models
5. **ML workflows** → Data Science notebooks

### Remember
- Fabric = All-in-one analytics platform
- OneLake = Unified data lake
- Lakehouse = Lake + Warehouse benefits
- Direct Lake = Fastest Power BI mode
- Medallion = Bronze (raw) → Silver (cleaned) → Gold (curated)
