# Core Data Concepts

## Overview
Understanding fundamental data concepts is essential for working with data in the cloud. This section covers data types, storage patterns, processing methods, and analytics techniques that form the foundation of Azure data services.

## Data Representation Types

### Structured Data
- **Definition:** Data organized in a predefined schema (tables with rows and columns)
- **Characteristics:**
  - Fixed schema with defined relationships
  - Easy to query using SQL
  - Maintains data integrity through constraints
- **Common Formats:**
  - Relational databases (SQL Server, PostgreSQL, MySQL)
  - CSV files with consistent columns
  - Excel spreadsheets
- **Use Cases:**
  - Transactional systems (e.g., e-commerce, banking)
  - Business applications requiring data consistency
  - Reporting and analytics with fixed schemas

### Semi-Structured Data
- **Definition:** Data with some organizational properties but no rigid schema
- **Characteristics:**
  - Self-describing structure with metadata tags
  - Flexible schema allowing variations
  - Hierarchical or nested organization
- **Common Formats:**
  - JSON (JavaScript Object Notation)
  - XML (eXtensible Markup Language)
  - YAML
  - Avro, Parquet, ORC
- **Use Cases:**
  - API responses and web services
  - Configuration files
  - Log files with varying structures
  - Data exchange between systems

### Unstructured Data
- **Definition:** Data without a predefined structure or organization
- **Characteristics:**
  - No fixed schema or data model
  - Requires processing to extract meaning
  - Largest volume of enterprise data
- **Common Formats:**
  - Documents (PDF, Word, text files)
  - Images (JPEG, PNG, GIF)
  - Videos and audio files
  - Social media posts
  - Email messages
- **Use Cases:**
  - Document management systems
  - Media libraries
  - Social media analytics
  - Machine learning training data

## Data Storage Options

### File Storage
- **Characteristics:**
  - Hierarchical directory structure
  - Store files of any type and size
  - Accessed via file system protocols (SMB, NFS)
- **Azure Services:**
  - Azure Blob Storage (object storage)
  - Azure Files (file shares)
  - Azure Data Lake Storage Gen2

### Databases
- **Relational Databases:**
  - Tables with defined relationships
  - ACID transactions (Atomicity, Consistency, Isolation, Durability)
  - SQL for querying and manipulation
  - Azure SQL Database, Azure Database for PostgreSQL/MySQL

- **Non-Relational Databases (NoSQL):**
  - Key-value stores: Simple lookup by key
  - Document databases: JSON-like documents
  - Column-family databases: Wide-column stores
  - Graph databases: Nodes and relationships
  - Azure Cosmos DB, Azure Table Storage

### Data Lakes
- **Characteristics:**
  - Store massive amounts of raw data in native format
  - Schema-on-read approach
  - Support structured, semi-structured, and unstructured data
  - Designed for big data analytics
- **Azure Services:**
  - Azure Data Lake Storage Gen2
  - Integrated with Azure Synapse Analytics and Azure Databricks

## Data Processing Solutions

### Batch Processing
- **Definition:** Processing large volumes of data in scheduled batches
- **Characteristics:**
  - Processes data at specific intervals (hourly, daily, weekly)
  - Handles large datasets efficiently
  - Non-real-time results
  - Lower cost per transaction
- **Common Scenarios:**
  - End-of-day financial reports
  - Monthly payroll processing
  - Data warehouse ETL jobs
  - Log file analysis
- **Azure Services:**
  - Azure Data Factory
  - Azure Synapse Analytics
  - Azure Databricks

### Stream Processing (Real-Time)
- **Definition:** Processing data continuously as it arrives
- **Characteristics:**
  - Immediate or near-real-time processing
  - Handles continuous data streams
  - Low-latency results
  - Higher infrastructure requirements
- **Common Scenarios:**
  - IoT sensor data processing
  - Fraud detection
  - Real-time dashboards
  - Social media sentiment analysis
  - Stock market analysis
- **Azure Services:**
  - Azure Stream Analytics
  - Azure Event Hubs
  - Azure IoT Hub

### ETL vs ELT

**ETL (Extract, Transform, Load):**
- Transform data before loading into destination
- Traditional approach for data warehouses
- Processing happens in a separate ETL tool
- Best for: Structured data, predefined transformations

**ELT (Extract, Load, Transform):**
- Load raw data first, transform later
- Modern approach for cloud data lakes
- Leverages processing power of destination system
- Best for: Big data, flexible analysis requirements

## Analytics Techniques

### Descriptive Analytics
- **Purpose:** What happened?
- **Description:** Analyzes historical data to understand past events
- **Techniques:**
  - Data aggregation and summarization
  - Reporting and dashboards
  - KPI tracking
- **Examples:**
  - Monthly sales reports
  - Website traffic statistics
  - Customer demographics

### Diagnostic Analytics
- **Purpose:** Why did it happen?
- **Description:** Examines data to identify causes and patterns
- **Techniques:**
  - Drill-down analysis
  - Data mining
  - Correlation analysis
- **Examples:**
  - Root cause analysis of sales decline
  - Identifying factors affecting customer churn
  - Performance bottleneck investigation

### Predictive Analytics
- **Purpose:** What will happen?
- **Description:** Uses statistical models and machine learning to forecast future outcomes
- **Techniques:**
  - Regression analysis
  - Time series forecasting
  - Machine learning models
- **Examples:**
  - Sales forecasting
  - Customer lifetime value prediction
  - Demand planning

### Prescriptive Analytics
- **Purpose:** What should we do?
- **Description:** Recommends actions based on analysis and predictions
- **Techniques:**
  - Optimization algorithms
  - Simulation modeling
  - Decision analysis
- **Examples:**
  - Inventory optimization recommendations
  - Dynamic pricing strategies
  - Resource allocation suggestions

## Data Visualization Concepts

### Key Principles
- **Choose the Right Chart Type:**
  - Bar/Column charts: Comparisons
  - Line charts: Trends over time
  - Pie charts: Proportions (limited use)
  - Scatter plots: Correlations
  - Heat maps: Patterns in large datasets

- **Design Best Practices:**
  - Keep visualizations simple and uncluttered
  - Use appropriate color schemes
  - Label axes and provide context
  - Highlight important insights
  - Ensure accessibility

### Azure Visualization Tools
- **Power BI:** Business intelligence and reporting
- **Azure Synapse Analytics:** Built-in visualization capabilities
- **Azure Databricks:** Data exploration and visualization notebooks

## Common Data Formats

### CSV (Comma-Separated Values)
- Simple text format with comma delimiters
- Easy to read and write
- Limited to flat, tabular data
- No standardized handling of special characters

### JSON (JavaScript Object Notation)
- Human-readable text format
- Supports hierarchical data structures
- Native format for web APIs
- Widely supported across platforms

### XML (eXtensible Markup Language)
- Markup language with custom tags
- Self-describing and hierarchical
- More verbose than JSON
- Strong in enterprise integration scenarios

### Avro
- Compact binary format
- Includes schema with data
- Supports schema evolution
- Good for big data processing

### Parquet
- Columnar storage format
- Highly compressed and efficient
- Optimized for analytics workloads
- Used in big data ecosystems (Hadoop, Spark)

### ORC (Optimized Row Columnar)
- Columnar storage format
- Excellent compression
- Built-in indexes for faster queries
- Popular in Hadoop environments

## Best Practices

### Data Storage Selection
1. **Consider data structure:**
   - Structured data with relationships → Relational database
   - Flexible schema requirements → NoSQL database
   - Large files and objects → Blob storage
   - Big data analytics → Data lake

2. **Evaluate access patterns:**
   - Transactional workloads → OLTP databases
   - Analytical workloads → OLAP/data warehouses
   - Archive and backup → Cool/archive storage tiers

3. **Plan for scale:**
   - Estimate data volume growth
   - Consider query performance requirements
   - Budget for storage and compute costs

### Processing Method Selection
1. **Choose batch processing when:**
   - Data can be processed in scheduled intervals
   - Real-time results are not required
   - Cost efficiency is a priority
   - Processing large historical datasets

2. **Choose stream processing when:**
   - Immediate insights are critical
   - Data arrives continuously
   - Real-time decision-making is needed
   - Low-latency requirements

## Study Tips

### Key Concepts to Master
- Differences between structured, semi-structured, and unstructured data
- When to use relational vs non-relational databases
- Batch vs stream processing scenarios
- Four types of analytics (descriptive, diagnostic, predictive, prescriptive)
- Common data formats and their use cases

### Practice Scenarios
1. Given a business requirement, identify the appropriate data storage type
2. Determine whether batch or stream processing is suitable
3. Match data formats to use cases
4. Identify the type of analytics being performed in scenarios

### Common Exam Questions
- Distinguishing between data types based on descriptions
- Selecting appropriate storage solutions for scenarios
- Identifying processing methods for requirements
- Understanding when to use different analytics techniques

### Remember
- Focus on concepts and when to use different approaches
- Understand the characteristics of each data type and storage option
- Know the differences between processing methods
- Be able to identify analytics types from scenario descriptions
