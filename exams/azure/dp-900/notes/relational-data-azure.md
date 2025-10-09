# Relational Data on Azure

## Overview
Relational databases organize data in tables with defined relationships, ensuring data integrity through constraints and supporting ACID transactions. Azure provides multiple fully managed relational database services for different scenarios and workloads.

## Relational Database Concepts

### Fundamental Components

#### Tables
- **Structure:** Rows and columns format
- **Rows (Records/Tuples):** Individual data entries
- **Columns (Attributes/Fields):** Data properties with specific data types
- **Schema:** Defines table structure, data types, and constraints
- **Example:**
  ```
  Customers Table:
  CustomerID | Name         | Email              | City
  -----------|--------------|--------------------|---------
  1          | John Smith   | john@email.com     | Seattle
  2          | Jane Doe     | jane@email.com     | Portland
  ```

#### Keys
- **Primary Key:**
  - Uniquely identifies each row in a table
  - Cannot be null
  - Only one primary key per table
  - Example: CustomerID, EmployeeID

- **Foreign Key:**
  - References primary key in another table
  - Establishes relationships between tables
  - Enforces referential integrity
  - Example: CustomerID in Orders table referencing Customers table

- **Composite Key:**
  - Primary key consisting of multiple columns
  - Example: OrderID + ProductID in OrderDetails table

- **Candidate Key:**
  - Columns that could serve as primary key
  - Unique and not null

- **Surrogate Key:**
  - Artificially created key (auto-increment ID)
  - Has no business meaning

#### Indexes
- **Purpose:** Speed up data retrieval operations
- **Types:**
  - Clustered Index: Determines physical order of data (one per table)
  - Non-Clustered Index: Separate structure with pointers to data (multiple allowed)
  - Unique Index: Ensures no duplicate values
  - Composite Index: Index on multiple columns

- **Trade-offs:**
  - Improves SELECT performance
  - Slows down INSERT, UPDATE, DELETE operations
  - Requires additional storage space

### Relationships

#### One-to-One (1:1)
- Each row in Table A relates to one row in Table B
- Example: Employee ↔ EmployeeDetails

#### One-to-Many (1:N)
- Each row in Table A can relate to multiple rows in Table B
- Most common relationship type
- Example: Customer → Orders (one customer, many orders)

#### Many-to-Many (M:N)
- Multiple rows in Table A relate to multiple rows in Table B
- Requires junction/bridge table
- Example: Students ↔ Courses (through Enrollments table)

### Constraints

#### Primary Key Constraint
- Enforces uniqueness and non-null
- Automatically creates unique index

#### Foreign Key Constraint
- Maintains referential integrity
- Prevents orphaned records
- Can specify CASCADE operations (ON DELETE CASCADE, ON UPDATE CASCADE)

#### Unique Constraint
- Ensures no duplicate values in column(s)
- Allows one NULL value

#### Check Constraint
- Enforces domain integrity
- Validates data based on logical expression
- Example: Age >= 18, Price > 0

#### Not Null Constraint
- Ensures column always has a value
- Prevents NULL entries

#### Default Constraint
- Provides default value if none specified
- Example: OrderDate defaults to current date

### Normalization

#### Purpose
- Minimize data redundancy
- Avoid update anomalies
- Ensure data consistency

#### Normal Forms

**First Normal Form (1NF):**
- Eliminate repeating groups
- Each column contains atomic (indivisible) values
- Each row is unique

**Second Normal Form (2NF):**
- Must be in 1NF
- All non-key attributes fully depend on primary key
- Eliminates partial dependencies

**Third Normal Form (3NF):**
- Must be in 2NF
- No transitive dependencies
- Non-key attributes depend only on primary key

#### Denormalization
- Intentionally introducing redundancy
- Improves query performance
- Common in data warehouses and reporting databases
- Trade-off: Performance vs consistency

## SQL Language Basics

### Data Definition Language (DDL)
- **CREATE:** Create database objects (tables, views, indexes)
- **ALTER:** Modify existing database objects
- **DROP:** Delete database objects
- **TRUNCATE:** Remove all rows from table

### Data Manipulation Language (DML)
- **SELECT:** Retrieve data from database
- **INSERT:** Add new rows to table
- **UPDATE:** Modify existing rows
- **DELETE:** Remove rows from table

### Data Control Language (DCL)
- **GRANT:** Give user permissions
- **REVOKE:** Remove user permissions

### Transaction Control Language (TCL)
- **BEGIN TRANSACTION:** Start a transaction
- **COMMIT:** Save changes permanently
- **ROLLBACK:** Undo changes
- **SAVEPOINT:** Create checkpoint within transaction

### Common SQL Operations

```sql
-- SELECT with filtering
SELECT CustomerName, City
FROM Customers
WHERE Country = 'USA'
ORDER BY CustomerName;

-- JOIN operations
SELECT o.OrderID, c.CustomerName, o.OrderDate
FROM Orders o
INNER JOIN Customers c ON o.CustomerID = c.CustomerID;

-- Aggregation
SELECT Country, COUNT(*) as CustomerCount
FROM Customers
GROUP BY Country
HAVING COUNT(*) > 5;

-- Subquery
SELECT ProductName
FROM Products
WHERE ProductID IN (SELECT ProductID FROM OrderDetails WHERE Quantity > 10);
```

## Azure Relational Database Services

### Azure SQL Database

#### Overview
- Fully managed PaaS SQL Server database
- Built-in high availability, backups, and patching
- Serverless and provisioned compute options
- Intelligent performance optimization

#### Key Features
- **Automatic tuning:** AI-powered performance recommendations
- **Elastic pools:** Share resources across multiple databases
- **Geo-replication:** Active geo-replication for disaster recovery
- **Scaling:** Scale compute and storage independently
- **Backup:** Automated backups with point-in-time restore

#### Deployment Options
- **Single Database:** Isolated database with dedicated resources
- **Elastic Pool:** Collection of databases sharing resources
- **Hyperscale:** Up to 100 TB databases with fast scaling

#### Service Tiers
- **General Purpose:** Balanced compute and storage (99.99% SLA)
- **Business Critical:** High performance with read replicas (99.995% SLA)
- **Hyperscale:** Large databases with dynamic scaling

#### Use Cases
- New cloud applications
- Modernizing existing applications
- SaaS applications
- Applications requiring high availability

### Azure SQL Managed Instance

#### Overview
- Near 100% SQL Server compatibility
- Instance-level features (SQL Agent, CLR, linked servers)
- Lift-and-shift migrations from on-premises
- Managed service with automated maintenance

#### Key Features
- **Native Virtual Network (VNet) support**
- **SQL Server Agent for job scheduling**
- **Cross-database queries**
- **Native backup/restore from on-premises**
- **Instance pools for smaller databases**

#### Service Tiers
- **General Purpose:** Budget-friendly, standard performance
- **Business Critical:** High IOPS, built-in read replicas

#### Use Cases
- Migrating SQL Server workloads to cloud
- Applications requiring instance-level features
- Multi-database applications
- Hybrid cloud scenarios

### SQL Server on Azure VMs

#### Overview
- Full control over SQL Server installation
- IaaS solution with OS-level access
- Any SQL Server version and edition
- Bring your own license (BYOL)

#### Key Features
- **Complete SQL Server access:** All features available
- **Custom configurations:** Control OS and SQL Server settings
- **Existing SQL Server licenses:** Azure Hybrid Benefit
- **Backup options:** Azure Backup or custom solutions
- **High availability:** Always On Availability Groups, Failover Cluster Instances

#### Use Cases
- Applications requiring specific SQL Server versions
- OS-level access requirements
- Custom SQL Server configurations
- Lift-and-shift with minimal changes

### Azure Database for PostgreSQL

#### Overview
- Fully managed PostgreSQL database service
- Community PostgreSQL on Azure
- Built-in high availability and backups

#### Deployment Options
- **Single Server:** Managed service with vertical scaling
- **Flexible Server:** More control over availability, scaling, and configuration
- **Hyperscale (Citus):** Distributed PostgreSQL for multi-tenant apps

#### Key Features
- **Extensions support:** PostGIS, pg_cron, timescaleDB
- **Connection pooling:** Built-in with Flexible Server
- **Intelligent performance:** Query Store and recommendations
- **Security:** Advanced threat protection, encryption

#### Use Cases
- Open-source PostgreSQL applications
- Geospatial applications (with PostGIS)
- Time-series data workloads
- Multi-tenant SaaS applications (Hyperscale)

### Azure Database for MySQL

#### Overview
- Fully managed MySQL database service
- Community MySQL on Azure
- Compatible with existing MySQL applications

#### Deployment Options
- **Single Server:** Standard managed service (legacy)
- **Flexible Server:** Enhanced flexibility and control (recommended)

#### Key Features
- **MySQL versions 5.7 and 8.0**
- **Automatic backups with point-in-time restore**
- **Scaling:** Compute and storage scaling
- **Read replicas:** Up to 5 read replicas
- **Advanced threat protection**

#### Use Cases
- PHP/MySQL web applications (LAMP stack)
- Migrating MySQL workloads to cloud
- WordPress and other CMS platforms
- E-commerce applications

### Azure Database for MariaDB

#### Overview
- Fully managed MariaDB database service
- Based on MariaDB community edition
- Compatible with MySQL applications

#### Key Features
- **Built-in high availability**
- **Automatic backups**
- **Read replicas for scaling**
- **Advanced security features**

#### Use Cases
- Applications using MariaDB
- MySQL alternative for open-source preference
- Applications requiring MariaDB-specific features

## Data Security and Compliance

### Authentication

#### Azure AD Authentication
- Centralized identity management
- Multi-factor authentication (MFA)
- Conditional access policies
- No password storage in database

#### SQL Authentication
- Traditional username/password
- Database-level authentication
- Stored in database
- Less secure than Azure AD

#### Windows Authentication
- Available for SQL Server on VMs
- Active Directory integration
- Kerberos authentication

### Authorization

#### Role-Based Access Control (RBAC)
- Azure-level permissions (control plane)
- Roles: Owner, Contributor, Reader
- Applied at subscription, resource group, or resource level

#### Database Roles
- Server-level roles (logins)
- Database-level roles (users)
- Fixed database roles: db_owner, db_datareader, db_datawriter
- Custom roles for specific permissions

#### Row-Level Security (RLS)
- Filter rows based on user identity
- Implemented using security policies
- Transparent to applications

#### Dynamic Data Masking
- Obfuscate sensitive data for non-privileged users
- Applied at database level
- Doesn't modify actual data
- Common patterns: Default, Email, Random, Custom

### Encryption

#### Encryption at Rest
- **Transparent Data Encryption (TDE):**
  - Encrypts database, backups, and logs
  - Enabled by default in Azure
  - Automatic and transparent to applications
  - Service-managed or customer-managed keys (CMK)

- **Always Encrypted:**
  - Encrypts specific columns
  - Data encrypted in client application
  - Database never sees unencrypted data
  - Requires driver support

#### Encryption in Transit
- **TLS/SSL encryption:**
  - Enabled by default
  - Protects data during transmission
  - Enforced connections option available

#### Key Management
- **Azure Key Vault integration:**
  - Store encryption keys securely
  - Centralized key management
  - Audit key usage
  - Customer-managed keys (CMK) option

### Compliance Standards

#### Common Standards
- **GDPR:** General Data Protection Regulation (EU)
- **HIPAA:** Health Insurance Portability and Accountability Act (US healthcare)
- **PCI DSS:** Payment Card Industry Data Security Standard
- **SOC 1/2/3:** Service Organization Control reports
- **ISO 27001:** Information security management

#### Azure Compliance
- Azure compliance certifications
- Shared responsibility model
- Compliance documentation and reports
- Microsoft Purview for compliance management

### Threat Protection

#### Microsoft Defender for SQL
- Threat detection and alerts
- Vulnerability assessment
- Security recommendations
- Monitors for suspicious activities

#### Firewall Rules
- Server-level firewall rules
- Database-level firewall rules (SQL Database only)
- Virtual Network rules
- Private endpoints for isolated connectivity

#### Auditing
- Track database events
- Write audit logs to Storage Account, Log Analytics, or Event Hub
- Compliance and security monitoring
- Threat detection integration

## Best Practices

### Design Best Practices
1. **Use appropriate data types** to save storage and improve performance
2. **Normalize for transactional systems**, denormalize for reporting
3. **Create indexes strategically** on frequently queried columns
4. **Use constraints** to enforce data integrity at database level
5. **Design for scale** with proper partitioning and sharding strategies

### Security Best Practices
1. **Use Azure AD authentication** whenever possible
2. **Enable TDE** for encryption at rest
3. **Enforce SSL/TLS** for connections
4. **Implement least privilege** access control
5. **Enable auditing and threat detection**
6. **Use private endpoints** for network isolation
7. **Regularly review** access permissions and firewall rules

### Performance Best Practices
1. **Monitor query performance** using Query Store
2. **Use connection pooling** to reduce overhead
3. **Implement caching** for frequently accessed data
4. **Optimize indexes** based on query patterns
5. **Use read replicas** for read-heavy workloads
6. **Scale appropriately** based on workload requirements

## Study Tips

### Key Differences to Know
- **SQL Database vs Managed Instance vs SQL VM:**
  - SQL Database: Highest abstraction, best for new apps
  - Managed Instance: SQL Server compatibility, instance features
  - SQL VM: Full control, any SQL Server version

- **Deployment model choice:**
  - PaaS for management simplicity
  - IaaS for control and customization

### Common Scenarios
1. Migrating on-premises SQL Server → Consider Managed Instance
2. New cloud-native application → Consider SQL Database
3. Open-source application → PostgreSQL or MySQL
4. Need OS-level access → SQL Server on VM

### Exam Focus Areas
- Understand when to use each Azure SQL service
- Know relational database fundamentals (keys, indexes, relationships)
- Understand normalization concepts
- Know security features (authentication, encryption, compliance)
- Understand basic SQL syntax and operations

### Practice Questions
- Given a scenario, which Azure SQL service is most appropriate?
- What type of relationship exists between tables?
- Which encryption method protects data at rest?
- What's the difference between authentication and authorization?
