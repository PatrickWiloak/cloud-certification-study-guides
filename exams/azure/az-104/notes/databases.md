# Azure Databases - AZ-104

## Azure SQL Database
- PaaS relational database
- Always latest SQL Server version
- Automatic patching and backups
- Built-in high availability

**Deployment Options**:
- Single database: Dedicated resources
- Elastic pool: Share resources across databases
- Managed instance: Near 100% SQL Server compatibility

**Service Tiers**:
- DTU-based: Basic, Standard, Premium
- vCore-based: General Purpose, Business Critical, Hyperscale

**Backup**:
- Automatic backups (7-35 days retention)
- Point-in-time restore
- Long-term retention (up to 10 years)
- Geo-replication for disaster recovery

## Azure Cosmos DB
- Globally distributed NoSQL
- Multi-model (document, key-value, graph, column-family)
- APIs: SQL, MongoDB, Cassandra, Gremlin, Table
- Multi-region writes
- Consistency levels: Strong, Bounded staleness, Session, Consistent prefix, Eventual

**Throughput**:
- Provisioned: Reserve RU/s
- Serverless: Pay per request
- Autoscale: Dynamic scaling

## Azure Database for MySQL/PostgreSQL
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
