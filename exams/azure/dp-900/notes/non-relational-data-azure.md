# Non-Relational Data on Azure

## Overview
Non-relational databases (NoSQL) provide flexible schemas and horizontal scaling for modern applications. Azure offers multiple NoSQL database services optimized for different data models and access patterns.

## Non-Relational Data Concepts

### Key-Value Stores

#### Characteristics
- Simplest NoSQL data model
- Data stored as key-value pairs
- Keys are unique identifiers
- Values can be any data type (string, number, JSON, binary)
- No schema enforcement
- Fast lookups by key

#### Use Cases
- Session management and caching
- User preferences and profiles
- Shopping carts
- Real-time recommendations
- Configuration management

#### Azure Services
- **Azure Table Storage:** Simple key-value store
- **Azure Cosmos DB (Table API):** Globally distributed key-value store
- **Azure Cache for Redis:** In-memory key-value store

#### Example
```
Key: user:1001
Value: {
  "name": "John Doe",
  "email": "john@example.com",
  "preferences": {"theme": "dark"}
}
```

### Document Databases

#### Characteristics
- Store data as documents (typically JSON)
- Self-describing with flexible schema
- Documents can have different structures
- Support nested objects and arrays
- Query within document properties
- Collections group related documents

#### Use Cases
- Content management systems
- User profiles with varying attributes
- Product catalogs with diverse properties
- Mobile app backends
- Event logging and IoT data

#### Azure Services
- **Azure Cosmos DB (NoSQL API, MongoDB API):** Primary document database service

#### Example
```json
{
  "id": "product-123",
  "name": "Wireless Mouse",
  "category": "Electronics",
  "price": 29.99,
  "specs": {
    "connectivity": "Bluetooth",
    "battery": "Rechargeable"
  },
  "reviews": [
    {"user": "john", "rating": 5},
    {"user": "jane", "rating": 4}
  ]
}
```

### Column-Family Databases

#### Characteristics
- Data organized in column families (groups of columns)
- Wide-column store format
- Rows can have different columns
- Optimized for reading/writing specific columns
- Efficient for sparse data
- High write throughput

#### Use Cases
- Time-series data
- IoT sensor data
- Event logging and auditing
- Real-time analytics
- Recommendation engines

#### Azure Services
- **Azure Cosmos DB (Cassandra API):** Wide-column store

#### Example Structure
```
RowKey: sensor-001-2024-01-15
Column Family: Measurements
  - temperature: 72.5
  - humidity: 45.2
  - timestamp: 2024-01-15T10:30:00Z

RowKey: sensor-001-2024-01-15-10:31
Column Family: Measurements
  - temperature: 72.7
  - pressure: 1013.25
```

### Graph Databases

#### Characteristics
- Data represented as nodes (entities) and edges (relationships)
- Optimized for traversing relationships
- Relationships are first-class citizens
- Support complex relationship queries
- Property graphs (nodes and edges have properties)

#### Use Cases
- Social networks and connections
- Recommendation engines
- Fraud detection
- Knowledge graphs
- Network and IT operations
- Organization hierarchies

#### Azure Services
- **Azure Cosmos DB (Gremlin API):** Graph database service

#### Example
```
Nodes:
  - Person(id: "p1", name: "Alice")
  - Person(id: "p2", name: "Bob")
  - Company(id: "c1", name: "TechCorp")

Edges:
  - p1 -[KNOWS]-> p2 (since: 2020)
  - p1 -[WORKS_AT]-> c1 (role: "Engineer")
  - p2 -[WORKS_AT]-> c1 (role: "Manager")
```

### Time Series Databases

#### Characteristics
- Optimized for time-stamped data
- Data points ordered by time
- High-volume ingestion
- Efficient storage with compression
- Time-based queries and aggregations
- Retention policies for data lifecycle

#### Use Cases
- IoT sensor monitoring
- Application performance monitoring (APM)
- Financial market data
- Server and infrastructure metrics
- Real-time analytics dashboards

#### Azure Services
- **Azure Time Series Insights:** Purpose-built for IoT time series
- **Azure Data Explorer:** Fast analytics on large time series datasets
- **Azure Cosmos DB:** Can handle time-series workloads

## Data Consistency Models

### Consistency Levels

#### Strong Consistency
- **Guarantee:** Reads always return the most recent write
- **Behavior:** All replicas synchronized before acknowledgment
- **Trade-offs:**
  - Highest consistency
  - Lowest availability
  - Highest latency
- **Use Cases:** Financial transactions, inventory management

#### Eventual Consistency
- **Guarantee:** All replicas eventually converge to same value
- **Behavior:** Writes propagate asynchronously
- **Trade-offs:**
  - Lowest consistency guarantees
  - Highest availability
  - Lowest latency
- **Use Cases:** Social media posts, view counts, analytics

#### Bounded Staleness
- **Guarantee:** Reads may lag behind writes by specified amount (time or operations)
- **Behavior:** Consistency within defined bounds
- **Trade-offs:**
  - Predictable staleness
  - Better availability than strong consistency
  - Moderate latency
- **Use Cases:** Sports scores, stock quotes, collaborative editing

#### Session Consistency
- **Guarantee:** Read-your-own-writes within a session
- **Behavior:** Consistent within user session, eventual across sessions
- **Trade-offs:**
  - Good balance for most applications
  - High availability
  - Low latency
- **Use Cases:** E-commerce, web applications, user profiles

#### Consistent Prefix
- **Guarantee:** Reads never see out-of-order writes
- **Behavior:** Updates observed in order, but may be stale
- **Trade-offs:**
  - Order preserved
  - High availability
  - Low latency
- **Use Cases:** Message threads, conversation history

### CAP Theorem

#### The Triangle Trade-off
- **C (Consistency):** All nodes see same data at same time
- **A (Availability):** Every request receives a response
- **P (Partition Tolerance):** System continues despite network partitions

#### Reality
- Can only guarantee 2 out of 3
- Network partitions will happen (P is required)
- Must choose between C and A during partitions

#### Azure Cosmos DB Approach
- Offers multiple consistency levels
- Allows choosing trade-offs per application
- Default: Session consistency (balanced approach)

## Azure Non-Relational Data Services

### Azure Cosmos DB

#### Overview
- Globally distributed, multi-model database
- Turnkey global distribution with multi-region writes
- SLA-backed performance: latency, throughput, consistency, availability
- Automatic indexing of all data
- Multiple API models

#### APIs Available

**NoSQL API (Core SQL API):**
- Native API for Cosmos DB
- SQL-like query syntax
- Best performance and features
- Schema-agnostic JSON documents

**MongoDB API:**
- Wire protocol compatible with MongoDB
- Migrate MongoDB apps without code changes
- MongoDB query language support

**Cassandra API:**
- Wire protocol compatible with Apache Cassandra
- CQL (Cassandra Query Language) support
- Wide-column store model

**Gremlin API (Graph):**
- Property graph model
- Gremlin query language
- Traverse relationships efficiently

**Table API:**
- Key-value store
- Premium alternative to Azure Table Storage
- Better performance and global distribution

#### Key Features
- **Global Distribution:**
  - Multi-region replication
  - Multi-region writes
  - Automatic failover
  - Configurable per request

- **Partitioning:**
  - Automatic horizontal scaling
  - Partition key selection critical
  - Unlimited storage and throughput

- **Request Units (RUs):**
  - Normalized cost of operations
  - Throughput measured in RU/s
  - Provisioned or serverless options

- **Consistency Levels:**
  - 5 well-defined levels
  - Configurable per-request
  - Balance consistency vs performance

- **Change Feed:**
  - Log of all changes to containers
  - Real-time event processing
  - Enables microservices patterns

#### Pricing Models
- **Provisioned Throughput:**
  - Pay for RU/s provisioned
  - Standard or autoscale
  - Best for predictable workloads

- **Serverless:**
  - Pay per request (RU consumed)
  - No capacity planning
  - Best for intermittent workloads

#### Use Cases
- Globally distributed applications
- Web and mobile backends
- IoT and telemetry data
- Retail and e-commerce
- Gaming leaderboards
- Real-time personalization

### Azure Table Storage

#### Overview
- Simple key-value NoSQL store
- Part of Azure Storage account
- Schema-less design
- Cost-effective for large datasets
- Automatic scaling

#### Data Model
- **Tables:** Container for entities
- **Entities:** Rows with properties (key-value pairs)
- **Partition Key:** Logical grouping of entities
- **Row Key:** Unique identifier within partition
- **Properties:** Up to 252 properties per entity

#### Key Features
- **Massive scale:** Petabytes of data
- **Cost-effective:** Lower cost than Cosmos DB
- **REST API:** Simple HTTP-based access
- **No schema enforcement:** Flexible entity structures
- **Automatic indexing:** On PartitionKey and RowKey

#### Limitations
- Limited query capabilities compared to SQL
- No cross-partition transactions
- No relationships between entities
- Single region (not globally distributed)

#### Use Cases
- Structured non-relational data
- Fast lookups by key
- Logging and diagnostics data
- Large datasets with simple queries
- Cost-sensitive applications

### Azure Blob Storage

#### Overview
- Object storage for unstructured data
- Optimized for storing massive amounts of data
- Accessible via REST API
- Tiered storage for cost optimization

#### Blob Types

**Block Blobs:**
- Text and binary data
- Up to 190.7 TB per blob
- Optimized for upload and download
- Ideal for: Documents, images, videos, backups

**Append Blobs:**
- Optimized for append operations
- Ideal for: Logging, streaming data

**Page Blobs:**
- Random read/write operations
- Up to 8 TB
- Ideal for: Virtual hard disks (VHDs)

#### Storage Tiers
- **Hot:** Frequent access, higher storage cost, lower access cost
- **Cool:** Infrequent access (30+ days), stored for at least 30 days
- **Cold:** Rare access (90+ days), stored for at least 90 days
- **Archive:** Rarely accessed (180+ days), lowest storage cost, highest retrieval cost

#### Key Features
- **Lifecycle management:** Automatic tier transitions
- **Versioning:** Keep multiple versions of blobs
- **Soft delete:** Recover deleted blobs
- **Immutable storage:** WORM (Write Once, Read Many) compliance
- **Encryption:** At rest and in transit

#### Use Cases
- Serving images/documents to browsers
- Storing files for distributed access
- Streaming video and audio
- Data backup and archiving
- Data for analysis (data lakes)
- Disaster recovery

### Azure Files

#### Overview
- Fully managed file shares in the cloud
- SMB and NFS protocol support
- Can be mounted on cloud or on-premises
- Replace or supplement on-premises file servers

#### Key Features
- **Protocol Support:** SMB 2.1, SMB 3.0, NFS 4.1
- **Shared Access:** Multiple VMs can access simultaneously
- **Managed Service:** No server management needed
- **Snapshots:** Point-in-time file share snapshots
- **Azure File Sync:** Sync with Windows Server

#### Tiers
- **Premium:** SSD-backed, low latency, high throughput
- **Transaction Optimized:** Standard, general purpose
- **Hot:** Optimized for frequent access
- **Cool:** Cost-optimized for infrequent access

#### Use Cases
- Lift-and-shift applications needing file shares
- Replace on-premises file servers
- Centralized logging and diagnostics
- Development and debugging tools
- Configuration management

### Azure Cache for Redis

#### Overview
- Fully managed in-memory data store
- Based on open-source Redis
- Microsecond latency
- High throughput

#### Key Features
- **Data Structures:** Strings, lists, sets, sorted sets, hashes, streams
- **Caching Patterns:** Cache-aside, write-through, read-through
- **Pub/Sub:** Messaging capabilities
- **Geo-replication:** Multi-region support
- **Persistence:** Optional data persistence to disk

#### Tiers
- **Basic:** Single node, no SLA, dev/test
- **Standard:** Replicated cache, 99.9% SLA
- **Premium:** Enhanced features, clustering, persistence
- **Enterprise:** Redis Enterprise features, 99.99% SLA

#### Use Cases
- Application caching
- Session state storage
- Message broker
- Real-time leaderboards
- Rate limiting
- Job queuing

## Best Practices

### Choosing the Right Service

**Use Azure Cosmos DB when:**
- Need global distribution
- Require low-latency reads/writes
- Multiple data models needed
- Automatic scaling required
- Mission-critical applications

**Use Azure Table Storage when:**
- Simple key-value lookups
- Cost is primary concern
- Don't need global distribution
- Massive datasets with simple queries

**Use Azure Blob Storage when:**
- Storing large unstructured data
- Files, images, videos, backups
- Data lakes and archival
- Cost-effective storage needed

**Use Azure Cache for Redis when:**
- Need in-memory performance
- Caching frequently accessed data
- Session state management
- Real-time applications

### Design Best Practices

#### Partition Key Selection (Cosmos DB/Table Storage)
1. **Distribute queries evenly** across partitions
2. **Avoid hot partitions** with uneven access patterns
3. **Consider cardinality:** Enough unique values for scale
4. **Query patterns:** Include partition key in queries when possible
5. **Example good keys:** UserID, TenantID, DeviceID
6. **Example bad keys:** Date (creates hot partitions), Boolean values

#### Data Modeling
1. **Embed related data** in documents when possible
2. **Denormalize for read performance** (unlike relational)
3. **Use references** for large or frequently updated data
4. **Design for your query patterns**
5. **Consider data access frequency**

#### Performance Optimization
1. **Index optimization:** Customize indexing policies (Cosmos DB)
2. **Use appropriate consistency level** for requirements
3. **Implement caching** for frequently accessed data
4. **Batch operations** when possible
5. **Monitor and optimize RU consumption** (Cosmos DB)

#### Cost Optimization
1. **Choose right pricing model:** Provisioned vs serverless
2. **Use autoscale** for variable workloads
3. **Implement data lifecycle policies**
4. **Archive old data** to cheaper storage tiers
5. **Optimize partition key** to avoid hot partitions
6. **Review and adjust throughput** regularly

### Security Best Practices
1. **Use Azure AD authentication** when available
2. **Enable encryption** at rest and in transit
3. **Implement network isolation:** Private endpoints, VNets
4. **Use Managed Identities** for service authentication
5. **Apply principle of least privilege**
6. **Enable auditing and monitoring**
7. **Backup and disaster recovery planning**

## Study Tips

### Key Concepts to Master
- Understand different NoSQL data models and when to use each
- Know Azure Cosmos DB APIs and their use cases
- Understand consistency levels and trade-offs
- Differentiate between Azure Table Storage and Cosmos DB
- Know blob storage types and access tiers
- Understand partition key importance and selection

### Common Scenarios
1. **Global web application** → Cosmos DB with NoSQL API
2. **Simple logging with high volume** → Azure Table Storage
3. **File sharing across VMs** → Azure Files
4. **Large media storage** → Azure Blob Storage (Hot/Cool/Archive)
5. **Caching frequently accessed data** → Azure Cache for Redis
6. **Social network relationships** → Cosmos DB with Gremlin API
7. **IoT sensor data** → Cosmos DB or Azure Data Explorer

### Exam Focus Areas
- When to use each NoSQL data model
- Azure Cosmos DB features and capabilities
- Consistency levels and CAP theorem
- Differences between storage services
- Appropriate service selection for scenarios
- Basic concepts of partition keys and scaling

### Practice Questions
- Given a scenario, which NoSQL service is most appropriate?
- What consistency level provides read-your-own-writes?
- What's the difference between Azure Table Storage and Cosmos DB Table API?
- When should you use eventual consistency vs strong consistency?
- What blob storage tier is best for archival data?

### Remember
- NoSQL = flexibility and scale, not just "no SQL"
- Cosmos DB = globally distributed, multi-model, premium features
- Table Storage = simple, cost-effective key-value store
- Consistency is a spectrum, choose based on requirements
- Partition key selection is critical for performance
