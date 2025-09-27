# Azure Database Fundamentals

## Database Service Overview

### Azure Database Portfolio
| Service | Type | Use Case |
|---------|------|----------|
| **Azure SQL Database** | Relational (PaaS) | Modern applications, microservices |
| **Azure SQL Managed Instance** | Relational (PaaS) | Lift-and-shift migrations |
| **SQL Server on VMs** | Relational (IaaS) | Full control, legacy applications |
| **Azure Database for PostgreSQL** | Relational (PaaS) | Open-source applications |
| **Azure Database for MySQL** | Relational (PaaS) | Web applications, LAMP stack |
| **Azure Database for MariaDB** | Relational (PaaS) | MySQL alternative |
| **Azure Cosmos DB** | NoSQL (multi-model) | Global distribution, low latency |
| **Azure Cache for Redis** | In-memory cache | Caching, session storage |
| **Azure Synapse Analytics** | Data warehouse | Analytics, business intelligence |

### Database Decision Tree
```
Need relational database? → SQL Database family
Need global distribution? → Cosmos DB
Need data warehouse? → Synapse Analytics
Need caching? → Azure Cache for Redis
Need full SQL Server features? → SQL Managed Instance or VM
Need open-source database? → PostgreSQL, MySQL, MariaDB
```

## Azure SQL Database

### What is Azure SQL Database?
Fully managed relational database service based on the latest stable version of Microsoft SQL Server.

### Service Tiers
| Tier | Model | Use Case |
|------|-------|----------|
| **Basic** | DTU | Small databases, light workloads |
| **Standard** | DTU | General purpose applications |
| **Premium** | DTU | I/O intensive applications |
| **General Purpose** | vCore | Balanced compute and storage |
| **Business Critical** | vCore | Mission-critical applications |
| **Hyperscale** | vCore | Large databases (up to 100TB) |

### Purchasing Models
#### DTU (Database Transaction Unit)
- **Pre-configured**: Fixed compute, storage, and I/O bundle
- **Simple**: Easy to understand and provision
- **Limited scalability**: Predefined performance levels
- **Use case**: Simple applications with predictable workloads

#### vCore (Virtual Core)
- **Flexible**: Independently scale compute and storage
- **Granular control**: Choose specific CPU, memory, storage
- **Hybrid benefit**: Use existing SQL Server licenses
- **Use case**: Applications requiring specific resource configurations

### Compute Tiers
| Tier | Description | Use Case |
|------|-------------|----------|
| **Provisioned** | Dedicated compute resources | Predictable workloads |
| **Serverless** | Auto-scaling compute | Intermittent workloads |

### High Availability Options
#### Built-in High Availability
- **General Purpose**: Zone-redundant storage with automatic failover
- **Business Critical**: Always On availability groups with readable secondaries
- **Hyperscale**: Multiple replicas with fast failover

#### Geo-Replication
```sql
-- Create geo-replica
ALTER DATABASE mydatabase 
ADD SECONDARY ON SERVER 'secondary-server' 
WITH (ALLOW_CONNECTIONS = READ_ONLY);

-- Failover to secondary
ALTER DATABASE mydatabase FAILOVER;
```

### Backup and Recovery
| Backup Type | Frequency | Retention |
|-------------|-----------|-----------|
| **Full backup** | Weekly | Up to 35 days |
| **Differential backup** | 12-24 hours | Up to 35 days |
| **Transaction log backup** | 5-10 minutes | Up to 35 days |

#### Point-in-Time Restore
```bash
# Restore database to specific point in time
az sql db restore \
  --dest-name restoreddb \
  --name originaldb \
  --resource-group myRG \
  --server myserver \
  --time "2023-01-15T10:30:00"
```

### Security Features
- **Transparent Data Encryption (TDE)**: Automatic encryption at rest
- **Always Encrypted**: Client-side encryption with key management
- **Dynamic Data Masking**: Obfuscate sensitive data for non-privileged users
- **Row Level Security**: Control access to rows based on user context
- **SQL Database Auditing**: Track database events and write to audit log

#### Always Encrypted Example
```sql
-- Create column master key
CREATE COLUMN MASTER KEY CMK1
WITH (
    KEY_STORE_PROVIDER_NAME = 'AZURE_KEY_VAULT',
    KEY_PATH = 'https://vault.vault.azure.net/keys/CMK1/key-version'
);

-- Create column encryption key
CREATE COLUMN ENCRYPTION KEY CEK1
WITH VALUES (
    COLUMN_MASTER_KEY = CMK1,
    ALGORITHM = 'RSA_OAEP',
    ENCRYPTED_VALUE = 0x016E000001630075007200720065006E0074...
);

-- Create table with encrypted column
CREATE TABLE Customers (
    CustomerId INT IDENTITY PRIMARY KEY,
    SSN CHAR(11) ENCRYPTED WITH (
        COLUMN_ENCRYPTION_KEY = CEK1,
        ENCRYPTION_TYPE = DETERMINISTIC,
        ALGORITHM = 'AEAD_AES_256_CBC_HMAC_SHA_256'
    ),
    Name NVARCHAR(50)
);
```

## Azure SQL Managed Instance

### What is SQL Managed Instance?
Fully managed SQL Server instance in the cloud with near 100% compatibility with on-premises SQL Server.

### Key Differences from SQL Database
| Feature | SQL Database | SQL Managed Instance |
|---------|--------------|---------------------|
| **SQL Server compatibility** | Core features | Near 100% compatibility |
| **Instance-level features** | Limited | Full support |
| **Cross-database queries** | No | Yes |
| **SQL Agent** | No | Yes |
| **CLR** | No | Yes |
| **Linked servers** | No | Yes |

### Service Tiers
| Tier | Description | Use Case |
|------|-------------|----------|
| **General Purpose** | Balanced performance | Most workloads |
| **Business Critical** | High performance, readable replicas | Mission-critical applications |

### Deployment Options
- **Single instance**: Dedicated managed instance
- **Instance pools**: Shared resources for smaller instances
- **SQL Managed Instance link**: Hybrid connectivity with on-premises

### Migration Scenarios
```bash
# Assess migration readiness
# Use Azure Migrate or Data Migration Assistant

# Migrate using Azure Database Migration Service
az dms project create \
  --name myMigrationProject \
  --source-platform SQL \
  --target-platform SQLMI \
  --resource-group myRG \
  --service-name myDMS
```

## Azure Database for PostgreSQL

### What is Azure Database for PostgreSQL?
Fully managed PostgreSQL database service with high availability, security, and scaling.

### Deployment Options
| Option | Description | Use Case |
|--------|-------------|----------|
| **Single Server** | Simple deployment | Small to medium applications |
| **Flexible Server** | Enhanced control and flexibility | Production workloads |
| **Hyperscale (Citus)** | Distributed PostgreSQL | Large-scale applications |

### Flexible Server Features
- **Zone-redundant high availability**: Cross-zone failover
- **Burstable compute**: Cost-effective for variable workloads
- **Custom maintenance windows**: Schedule maintenance
- **Enhanced monitoring**: Better metrics and alerts

### PostgreSQL Extensions
```sql
-- Popular extensions available
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";    -- UUID generation
CREATE EXTENSION IF NOT EXISTS "postgis";      -- Geographic objects
CREATE EXTENSION IF NOT EXISTS "pg_stat_statements"; -- Query statistics
CREATE EXTENSION IF NOT EXISTS "pg_trgm";      -- Text similarity
```

### Configuration Examples
```bash
# Create Flexible Server
az postgres flexible-server create \
  --name mypostgresserver \
  --resource-group myRG \
  --location eastus \
  --admin-user myadmin \
  --admin-password mypassword \
  --sku-name Standard_B2s \
  --tier Burstable \
  --storage-size 32 \
  --version 13
```

## Azure Database for MySQL

### What is Azure Database for MySQL?
Fully managed MySQL database service based on MySQL Community Edition.

### Deployment Options
| Option | Description | MySQL Version |
|--------|-------------|---------------|
| **Single Server** | Legacy option | 5.6, 5.7, 8.0 |
| **Flexible Server** | Current recommendation | 5.7, 8.0 |

### MySQL-Specific Features
- **Read replicas**: Scale read workloads
- **Automatic backups**: Point-in-time recovery
- **Performance recommendations**: Query optimization suggestions
- **Connection pooling**: Efficient connection management

```bash
# Create MySQL Flexible Server
az mysql flexible-server create \
  --name mymysqlserver \
  --resource-group myRG \
  --location eastus \
  --admin-user myadmin \
  --admin-password mypassword \
  --sku-name Standard_B2s \
  --storage-size 20 \
  --version 8.0
```

## Azure Cosmos DB

### What is Cosmos DB?
Globally distributed, multi-model NoSQL database service with guaranteed low latency and elastic scale.

### API Models
| API | Description | Use Case |
|-----|-------------|----------|
| **Core (SQL)** | Document database with SQL queries | New applications |
| **MongoDB** | Compatible with MongoDB | Migrate MongoDB apps |
| **Cassandra** | Wide-column store | Big data applications |
| **Gremlin** | Graph database | Social networks, recommendations |
| **Table** | Key-value store | Simple lookup scenarios |

### Consistency Levels
| Level | Description | Use Case |
|-------|-------------|----------|
| **Strong** | Linearizable reads | Financial applications |
| **Bounded Staleness** | Consistent prefix with lag bounds | Global apps with some tolerance |
| **Session** | Consistent within client session | Most applications |
| **Consistent Prefix** | Reads never see out-of-order writes | Social media feeds |
| **Eventual** | Strongest performance | Analytics, logging |

### Partitioning
```javascript
// Good partition key examples
{
  "id": "user123",
  "userId": "user123",     // Good: evenly distributed
  "category": "electronics", // Bad: might create hot partitions
  "timestamp": "2023-01-15"  // Bad: sequential, creates hot partitions
}

// Synthetic partition key
{
  "id": "user123",
  "partitionKey": "user123_2023-01"  // Combine user and time period
}
```

### Request Units (RUs)
- **Throughput currency**: All operations measured in RUs
- **Provisioned**: Reserve specific RU/s capacity
- **Serverless**: Pay per request (good for variable workloads)
- **Autoscale**: Automatically scale between 10% and 100% of max RU/s

### Global Distribution
```bash
# Add region to Cosmos account
az cosmosdb update \
  --name mycosmosaccount \
  --resource-group myRG \
  --locations regionName=eastus failoverPriority=0 isZoneRedundant=false \
  --locations regionName=westus failoverPriority=1 isZoneRedundant=false
```

## Azure Cache for Redis

### What is Azure Cache for Redis?
Fully managed in-memory data store based on Redis software.

### Service Tiers
| Tier | Description | Use Case |
|------|-------------|----------|
| **Basic** | Single node, no SLA | Development, testing |
| **Standard** | Two nodes with replication | Production applications |
| **Premium** | Enhanced features (clustering, persistence) | Enterprise applications |

### Premium Features
- **Redis clustering**: Scale beyond single node limits
- **Data persistence**: RDB and AOF persistence options
- **Virtual Network**: Deploy in your VNet
- **Geo-replication**: Replicate across regions

### Common Use Cases
#### Caching
```python
import redis

# Connect to Azure Cache for Redis
r = redis.Redis(
    host='mycache.redis.cache.windows.net',
    port=6380,
    password='access-key',
    ssl=True
)

# Cache frequently accessed data
def get_user_profile(user_id):
    # Check cache first
    cached_profile = r.get(f"user_profile:{user_id}")
    if cached_profile:
        return json.loads(cached_profile)
    
    # Fetch from database if not in cache
    profile = fetch_from_database(user_id)
    
    # Cache for 1 hour
    r.setex(f"user_profile:{user_id}", 3600, json.dumps(profile))
    return profile
```

#### Session Storage
```python
# Store session data
session_data = {
    "user_id": "123",
    "username": "john_doe",
    "preferences": {"theme": "dark", "language": "en"}
}

r.hmset(f"session:{session_id}", session_data)
r.expire(f"session:{session_id}", 1800)  # 30 minutes
```

#### Real-time Analytics
```python
# Increment counters
r.incr("page_views:today")
r.hincrby("page_views:by_page", "/products", 1)

# Leaderboards
r.zadd("game_scores", {"player1": 1500, "player2": 1200})
top_players = r.zrevrange("game_scores", 0, 9, withscores=True)
```

## Azure Synapse Analytics

### What is Azure Synapse Analytics?
Analytics service that brings together data integration, data warehousing, and big data analytics.

### Synapse Components
| Component | Description | Use Case |
|-----------|-------------|----------|
| **SQL Pools** | Data warehouse compute | Structured data analytics |
| **Spark Pools** | Big data processing | Data transformation, ML |
| **Pipelines** | Data integration | ETL/ELT workflows |
| **Link** | Real-time analytics | Operational analytics |

### SQL Pool (Data Warehouse)
#### Architecture
- **Massively Parallel Processing (MPP)**: Distribute queries across nodes
- **Distributions**: Data spread across 60 distributions
- **Compute nodes**: Process queries in parallel
- **Control node**: Orchestrates query execution

#### Performance Optimization
```sql
-- Create distributed table
CREATE TABLE FactSales (
    SalesKey INT NOT NULL,
    DateKey INT NOT NULL,
    ProductKey INT NOT NULL,
    SalesAmount DECIMAL(10,2)
)
WITH (
    DISTRIBUTION = HASH(ProductKey),
    CLUSTERED COLUMNSTORE INDEX
);

-- Create replicated dimension table
CREATE TABLE DimProduct (
    ProductKey INT NOT NULL,
    ProductName NVARCHAR(50),
    Category NVARCHAR(25)
)
WITH (
    DISTRIBUTION = REPLICATE,
    CLUSTERED COLUMNSTORE INDEX
);
```

### Spark Pools
```python
# PySpark example for data transformation
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, sum, avg

spark = SparkSession.builder.appName("DataTransformation").getOrCreate()

# Read data from Data Lake
df = spark.read.parquet("abfss://container@storage.dfs.core.windows.net/raw-data/")

# Transform data
aggregated_df = df.groupBy("product_category", "region") \
                  .agg(sum("sales_amount").alias("total_sales"),
                       avg("sales_amount").alias("avg_sales")) \
                  .filter(col("total_sales") > 10000)

# Write to Data Lake
aggregated_df.write.mode("overwrite") \
             .parquet("abfss://container@storage.dfs.core.windows.net/processed-data/")
```

## Database Migration

### Azure Database Migration Service (DMS)
**Fully managed service for database migrations**

#### Supported Migration Paths
| Source | Target | Migration Type |
|--------|--------|----------------|
| **SQL Server** | Azure SQL DB/MI | Online/Offline |
| **Oracle** | Azure SQL Database | Offline |
| **MySQL** | Azure Database for MySQL | Online/Offline |
| **PostgreSQL** | Azure Database for PostgreSQL | Online/Offline |
| **MongoDB** | Cosmos DB (MongoDB API) | Online/Offline |

#### Migration Process
```bash
# Create DMS instance
az dms create \
  --name mydms \
  --resource-group myRG \
  --location eastus \
  --sku-name Premium_4vCores

# Create migration project
az dms project create \
  --name myproject \
  --source-platform SQL \
  --target-platform SQLDB \
  --resource-group myRG \
  --service-name mydms
```

### Data Migration Tools
| Tool | Purpose | Use Case |
|------|---------|----------|
| **Azure Migrate** | Assessment and migration | Discover and assess workloads |
| **Data Migration Assistant** | SQL Server assessment | Compatibility analysis |
| **SQL Server Migration Assistant** | Heterogeneous migrations | Oracle, MySQL to SQL Server |
| **Azure Data Factory** | Data integration | ETL/ELT pipelines |

## Database Security

### Encryption
#### Transparent Data Encryption (TDE)
```sql
-- Enable TDE (enabled by default in Azure)
ALTER DATABASE mydatabase SET ENCRYPTION ON;

-- Check encryption status
SELECT 
    name,
    is_encrypted,
    encryption_state,
    percent_complete
FROM sys.dm_database_encryption_keys
JOIN sys.databases ON database_id = db_id();
```

#### Always Encrypted
- **Client-side encryption**: Data encrypted before sending to database
- **Key management**: Keys stored in Key Vault or certificate store
- **Query limitations**: Limited operations on encrypted columns

### Access Control
#### Azure AD Authentication
```sql
-- Create Azure AD user
CREATE USER [alice@company.com] FROM EXTERNAL PROVIDER;

-- Grant permissions
ALTER ROLE db_datareader ADD MEMBER [alice@company.com];
```

#### Row Level Security
```sql
-- Create security policy
CREATE FUNCTION dbo.userAccessPredicate(@UserId NVARCHAR(256))
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN SELECT 1 AS accessResult
WHERE @UserId = USER_NAME();

-- Apply policy to table
CREATE SECURITY POLICY dbo.userSecurityPolicy
ADD FILTER PREDICATE dbo.userAccessPredicate(UserId) ON dbo.UserData
WITH (STATE = ON);
```

### Auditing and Monitoring
```bash
# Enable SQL Database auditing
az sql db audit-policy update \
  --name mydatabase \
  --resource-group myRG \
  --server myserver \
  --state Enabled \
  --storage-account mystorage \
  --storage-endpoint https://mystorage.blob.core.windows.net \
  --storage-account-access-key "storage-key"
```

## Database Best Practices

### Performance Optimization
1. **Choose appropriate service tier**: Match performance to workload requirements
2. **Optimize queries**: Use indexes, avoid unnecessary data retrieval
3. **Monitor performance**: Use Query Performance Insight and Query Store
4. **Scale appropriately**: Use elastic pools for multiple databases
5. **Implement caching**: Use Azure Cache for Redis for frequently accessed data

### Security Best Practices
1. **Use Azure AD authentication**: Centralized identity management
2. **Enable encryption**: TDE for data at rest, SSL for data in transit
3. **Implement network isolation**: Use VNet integration and private endpoints
4. **Regular security updates**: Keep database engines updated
5. **Audit database access**: Enable auditing and monitor suspicious activities

### Cost Optimization
1. **Right-size resources**: Choose appropriate compute and storage sizes
2. **Use reserved capacity**: Commit to long-term usage for discounts
3. **Implement auto-scaling**: Scale resources based on demand
4. **Archive old data**: Move infrequently accessed data to cheaper storage
5. **Monitor usage**: Regular cost analysis and optimization

### Backup and Recovery
1. **Automated backups**: Enable automatic backup for all production databases
2. **Test recovery**: Regularly test backup and restore procedures
3. **Cross-region backups**: Implement geo-redundant backup for disaster recovery
4. **Document procedures**: Clear backup and recovery documentation
5. **Monitor backup health**: Ensure backups are completing successfully

## Common Database Pitfalls

### Performance Issues
- **Poor query design**: Missing indexes, inefficient queries
- **Wrong service tier**: Under or over-provisioned resources
- **Inadequate monitoring**: Not tracking performance metrics
- **Hot partitions**: Uneven data distribution in Cosmos DB
- **Connection pooling**: Not implementing proper connection management

### Security Vulnerabilities
- **Weak authentication**: Using SQL authentication instead of Azure AD
- **Overly permissive access**: Granting excessive database permissions
- **Unencrypted connections**: Not enforcing SSL/TLS
- **Missing auditing**: No audit trail for database access
- **Default configurations**: Not customizing security settings

### Operational Problems
- **Insufficient backups**: Inadequate backup strategies
- **No disaster recovery**: Missing cross-region recovery planning
- **Poor monitoring**: Insufficient alerting and monitoring
- **Manual scaling**: Not implementing auto-scaling for variable workloads
- **Lack of documentation**: Poor documentation of database configurations

### Cost Issues
- **Over-provisioning**: Allocating more resources than needed
- **Wrong pricing model**: Not choosing optimal pricing options
- **Unused resources**: Idle databases consuming resources
- **Data retention**: Keeping data longer than necessary
- **Missing optimization**: Not using reserved capacity or discounts