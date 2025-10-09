# Design and Implement Data Storage

## Overview
This domain covers designing data storage structures, partition strategies, and serving layers for analytics solutions. It represents 15-20% of the exam and focuses on foundational data architecture decisions that impact performance, cost, and maintainability.

## Key Topics

### Data Storage Structure Design
- Azure Data Lake solution design
- File type recommendations for storage
- File type recommendations for analytical queries
- Efficient querying design
- Data pruning design
- Folder structure for data transformation levels
- Distribution strategy design
- Data archiving solution design

### Partition Strategy Design
- Partition strategy for files
- Partition strategy for analytical workloads
- Partition strategy for efficiency and performance
- Partition strategy for Azure Synapse Analytics
- Partitioning needs in Azure Data Lake Storage Gen2

### Serving Layer Design
- Star schema design
- Slowly changing dimensions (SCD) implementation
- Dimensional hierarchy design
- Temporal data solution design
- Incremental loading design
- Analytical stores design
- Metastores in Azure Synapse Analytics and Databricks

## Services Reference

### Core Services
- Azure Data Lake Storage Gen2
- Azure Synapse Analytics (SQL Pools, Spark Pools)
- Azure Databricks
- Azure Blob Storage
- Parquet, Delta Lake, ORC file formats

### Supporting Services
- Azure Storage Explorer
- Azure Data Factory
- PolyBase
- Azure Purview (for data governance)

## Best Practices

### Data Lake Organization
- Implement medallion architecture (bronze, silver, gold layers)
- Use hierarchical namespace for efficient operations
- Organize data by domain, then by entity
- Include metadata in folder structure (year/month/day for time-series)
- Separate raw, processed, and curated data

### File Format Selection
- Use Parquet for analytical workloads (columnar, compressed)
- Use Delta Lake for ACID transactions and time travel
- Use ORC for highly compressed storage
- Avoid CSV/JSON for large analytical datasets
- Consider Avro for schema evolution scenarios

### Partitioning Strategies
- Partition by date for time-series data (most common)
- Avoid over-partitioning (too many small files)
- Aim for partition sizes of 256MB to 1GB
- Consider query patterns when designing partitions
- Use partitioning to enable partition pruning

### Data Warehouse Design
- Use star schema for simplicity and performance
- Implement SCD Type 2 for historical tracking
- Denormalize for query performance
- Use surrogate keys for dimension tables
- Design fact tables with appropriate grain

## Common Scenarios

### Data Lake Architecture
- Raw data ingestion (bronze layer)
- Data cleansing and transformation (silver layer)
- Business-ready aggregated data (gold layer)
- Multi-zone data lake with access controls
- Data lifecycle management and archival

### Partitioning Patterns
- Date-based partitioning for event logs
- Geographic partitioning for regional data
- Customer-based partitioning for multi-tenant systems
- Hash partitioning for even distribution
- Hybrid partitioning strategies

### Dimensional Modeling
- Customer dimension with SCD Type 2
- Date dimension with calendar hierarchies
- Product dimension with categories
- Sales fact table with grain at transaction level
- Accumulating snapshot fact tables

### Performance Optimization
- Partition pruning for query performance
- File compaction for small file problems
- Distribution keys for parallel processing
- Materialized views for common queries
- Table statistics for query optimization

## Study Tips

- Understand medallion architecture and its benefits
- Practice designing star schemas for different business scenarios
- Learn the three types of slowly changing dimensions (SCD Type 1, 2, 3)
- Hands-on experience with Parquet and Delta Lake formats
- Study partition size recommendations and impacts
- Practice folder structure design for Data Lake Gen2
- Understand distribution strategies in Synapse dedicated SQL pools
- Learn when to use clustered columnstore vs heap vs B-tree indexes
- Study temporal table patterns for historical data
- Practice incremental load patterns and change data capture
