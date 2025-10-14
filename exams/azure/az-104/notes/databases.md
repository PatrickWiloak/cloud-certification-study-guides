# Azure Databases - AZ-104

**[📖 Azure Databases Documentation](https://learn.microsoft.com/en-us/azure/?product=databases)** - Managed database services on Azure

## Azure SQL Database

**[📖 Azure SQL Database Documentation](https://learn.microsoft.com/en-us/azure/azure-sql/database/)** - Fully managed SQL Server database engine
- PaaS relational database
- Always latest SQL Server version
- Automatic patching and backups
- Built-in high availability

**Deployment Options**:

**[📖 Azure SQL Deployment Options](https://learn.microsoft.com/en-us/azure/azure-sql/azure-sql-iaas-vs-paas-what-is-overview)** - Choose the right SQL deployment model

- Single database: Dedicated resources
- Elastic pool: Share resources across databases
- Managed instance: Near 100% SQL Server compatibility

**Service Tiers**:

**[📖 SQL Database Service Tiers](https://learn.microsoft.com/en-us/azure/azure-sql/database/service-tiers-general-purpose-business-critical)** - Understand vCore and DTU purchasing models

- DTU-based: Basic, Standard, Premium
- vCore-based: General Purpose, Business Critical, Hyperscale

**Backup**:

**[📖 Automated Backups in SQL Database](https://learn.microsoft.com/en-us/azure/azure-sql/database/automated-backups-overview)** - Backup and restore capabilities

- Automatic backups (7-35 days retention)
- Point-in-time restore
- Long-term retention (up to 10 years)
- Geo-replication for disaster recovery

## Azure Cosmos DB

**[📖 Azure Cosmos DB Documentation](https://learn.microsoft.com/en-us/azure/cosmos-db/)** - Globally distributed, multi-model NoSQL database

- Globally distributed NoSQL
- Multi-model (document, key-value, graph, column-family)
- APIs: SQL, MongoDB, Cassandra, Gremlin, Table
- Multi-region writes
- Consistency levels: Strong, Bounded staleness, Session, Consistent prefix, Eventual

**Throughput**:

**[📖 Cosmos DB Request Units](https://learn.microsoft.com/en-us/azure/cosmos-db/request-units)** - Understand and optimize throughput costs

- Provisioned: Reserve RU/s
- Serverless: Pay per request
- Autoscale: Dynamic scaling

## Azure Database for MySQL/PostgreSQL

**[📖 Azure Database for MySQL](https://learn.microsoft.com/en-us/azure/mysql/)** - Managed MySQL database service

**[📖 Azure Database for PostgreSQL](https://learn.microsoft.com/en-us/azure/postgresql/)** - Managed PostgreSQL database service

- Fully managed open-source databases
- Deployment: Single server, Flexible server
- Automatic backups and patching
- High availability options
- Read replicas

## Exam Tips
- SQL Database for relational, PaaS
- Managed Instance for lift-and-shift SQL Server
- Cosmos DB for global, multi-region NoSQL
- Elastic pools for multiple databases with varying usage
- vCore for predictable workloads, DTU for simplicity
