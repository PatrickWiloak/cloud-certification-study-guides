# Azure Cosmos DB Fundamentals

## Overview
Azure Cosmos DB is a globally distributed, multi-model NoSQL database service designed for high availability, low latency, and elastic scalability.

## Data Models and APIs

### NoSQL API (Core SQL API)
- **Native API** for Cosmos DB
- **SQL-like queries** on JSON documents
- **Best performance** and features
- **Recommended** for new applications

### MongoDB API
- **MongoDB compatibility** (wire protocol compatible)
- **Migrate MongoDB apps** without code changes
- **MongoDB query language** support

### Cassandra API
- **Apache Cassandra compatible**
- **CQL (Cassandra Query Language)**
- **Wide-column store** model

### Gremlin API
- **Graph database** model
- **Apache TinkerPop** compatible
- **Gremlin query language**
- **Best for:** Social networks, recommendations

### Table API
- **Key-value store**
- **Upgrade** from Azure Table Storage
- **Better performance and global distribution**

## Partitioning Strategy

### Partition Key Selection
**Critical for performance and cost:**
- **High cardinality:** Many unique values
- **Even distribution:** Avoid hot partitions
- **Query patterns:** Include in queries when possible

**Examples:**
- **Good:** UserId, TenantId, DeviceId
- **Bad:** Country (low cardinality), Date (hot partitions)

### Hierarchical Partition Keys
- **Sub-partition** data for better distribution
- **Example:** `/TenantId/UserId`
- **Requires SDK support**

## Consistency Levels

### Strong
- **Linearizability** guarantee
- **Highest consistency, lowest availability**
- **Use:** Financial transactions, inventory

### Bounded Staleness
- **Reads lag by K versions or T time**
- **Predictable staleness**
- **Use:** Stock quotes, scoreboards

### Session
- **Read-your-own-writes**
- **Default** consistency level
- **Use:** Most applications

### Consistent Prefix
- **Reads never see out-of-order writes**
- **Use:** Social media updates

### Eventual
- **Lowest latency, highest availability**
- **Use:** View counts, non-critical data

## Request Units (RUs)

### RU Consumption
- **1 RU = Read 1KB item by ID and partition key**
- **Writes cost more** than reads
- **Queries cost** varies by complexity

### Provisioning Models

**Provisioned Throughput:**
- **Fixed RU/s** allocation
- **Standard** or **Autoscale**
- **Best for:** Predictable workloads

**Serverless:**
- **Pay-per-request**
- **No capacity planning**
- **Best for:** Intermittent workloads

## Global Distribution

### Multi-Region Writes
- **Write to any region**
- **Conflict resolution** policies
- **Last-Write-Wins** (default)
- **Custom conflict resolution**

### Automatic Failover
- **Transparent** to applications
- **Configurable priority** order
- **No downtime**

## Performance Optimization

### Indexing
- **Automatic indexing** of all properties
- **Customize** indexing policy
- **Exclude** large properties (reduce RU cost)

```json
{
  "indexingMode": "consistent",
  "automatic": true,
  "includedPaths": [{"path": "/*"}],
  "excludedPaths": [{"path": "/largeProperty/?"}]
}
```

### Change Feed
- **Real-time notification** of changes
- **Enable event-driven** architectures
- **Process changes** in order per partition
- **Use cases:** Materialized views, real-time analytics

## Best Practices

### Data Modeling
1. **Denormalize** for read performance
2. **Embed** related data when possible
3. **Reference** for large or frequently updated data
4. **Model for queries** not normalization

### Partition Key
1. **Avoid hot partitions**
2. **Include in queries** for efficiency
3. **Plan for growth**
4. **Use hierarchical** keys when needed

### Cost Optimization
1. **Right-size** RU/s provisioning
2. **Use TTL** for expiring data
3. **Optimize queries** to reduce RU consumption
4. **Archive old data** to cheaper storage

## Study Tips

### Key Concepts
- API selection (SQL, MongoDB, Cassandra, etc.)
- Partition key importance
- Consistency levels (5 levels)
- RU consumption and provisioning
- Global distribution capabilities

### Common Scenarios
1. **NoSQL document DB** → NoSQL API
2. **Migrate MongoDB** → MongoDB API
3. **Graph relationships** → Gremlin API
4. **Financial system** → Strong consistency
5. **Social media** → Eventual consistency
6. **E-commerce cart** → Session consistency

### Remember
- Partition key selection is critical
- Session consistency is default
- RU/s = throughput capacity
- Global distribution = multi-region writes
- Change Feed = event-driven architecture
