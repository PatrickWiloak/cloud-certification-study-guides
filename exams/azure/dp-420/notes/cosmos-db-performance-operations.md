# Cosmos DB Performance and Operations

## Table of Contents
1. [Request Units and Throughput](#request-units-and-throughput)
2. [Query Optimization](#query-optimization)
3. [Indexing Strategies](#indexing-strategies)
4. [Consistency Levels](#consistency-levels)
5. [Change Feed](#change-feed)
6. [Monitoring and Diagnostics](#monitoring-and-diagnostics)
7. [Backup and Disaster Recovery](#backup-and-disaster-recovery)
8. [Exam Tips](#exam-tips)

---

## Request Units and Throughput

### Understanding Request Units (RUs)

#### What is a Request Unit?
- Abstract measure of database operations
- Combines CPU, memory, and IOPS
- 1 RU = Read 1 KB item by ID and partition key
- Deterministic and predictable

#### RU Consumption Examples
| Operation | Approximate RU Cost |
|-----------|-------------------|
| Point read (1 KB) | 1 RU |
| Point read (100 KB) | 10 RU |
| Query (1 KB result) | 2.3+ RU |
| Insert (1 KB) | 5-6 RU |
| Update (1 KB) | 5-10 RU |
| Delete (1 KB) | 5 RU |
| Stored procedure | Varies by logic |

### Factors Affecting RU Consumption

#### 1. Item Size
- Larger items = more RUs
- Linear relationship
- Reading 10 KB ≈ 10 RUs (point read)

#### 2. Item Properties
- More properties = more indexing overhead
- Nested properties cost more
- Arrays and objects increase cost

#### 3. Indexing
- More indexes = higher write cost
- Excluding properties reduces cost
- Indexing policy impacts all operations

#### 4. Consistency Level
| Consistency | Read RU Cost | Write RU Cost |
|-------------|--------------|---------------|
| Strong | 2x | 1x |
| Bounded Staleness | 2x | 1x |
| Session | 1x | 1x |
| Consistent Prefix | 1x | 1x |
| Eventual | 0.5x | 1x |

#### 5. Query Complexity
- Filters and predicates
- Aggregations (COUNT, SUM, AVG)
- ORDER BY, GROUP BY
- JOINs within document
- User-defined functions (UDFs)

### Throughput Models

#### Provisioned Throughput (Manual)
```csharp
// Database-level (shared across containers)
await client.CreateDatabaseAsync(
    new DatabaseProperties { Id = "myDatabase" },
    throughput: 10000
);

// Container-level (dedicated)
await database.CreateContainerAsync(
    new ContainerProperties("myContainer", "/partitionKey"),
    throughput: 1000
);
```

**Characteristics:**
- Fixed RU/s allocation
- Reserved capacity
- Predictable cost
- Can scale up/down programmatically

**Cost:** $0.008 per 100 RU/s per hour (varies by region)

#### Autoscale Throughput
```csharp
await database.CreateContainerAsync(
    new ContainerProperties("myContainer", "/partitionKey"),
    ThroughputProperties.CreateAutoscaleThroughput(maxRU: 10000)
);
```

**Characteristics:**
- Scales from 0.1x to max (10% to 100%)
- Automatic scaling based on usage
- Scales up instantly, down gradually
- Pay for actual peak usage per hour

**When to Use:**
- Variable workloads
- Unpredictable traffic
- Spiky patterns
- Development/test environments

**Cost Calculation:**
- Minimum: 10% of max RU/s
- Charged hourly for highest RU/s used
- Example: Max 10K RU/s, used 5K → charged for 5K

#### Serverless
```csharp
// Account-level setting (serverless account)
// No throughput provisioning needed
await database.CreateContainerAsync(
    new ContainerProperties("myContainer", "/partitionKey")
);
```

**Characteristics:**
- Pay-per-request
- No minimum throughput
- Auto-scales to workload
- Container limits: 50 GB storage, 5K RU/s

**When to Use:**
- Development and testing
- Infrequent access patterns
- Small workloads
- Unpredictable traffic (small scale)

**Cost:** Per million RU consumed + storage

### Burst Capacity
Temporary ability to exceed provisioned throughput.

**Features:**
- Up to 300 seconds of burst
- System provides additional RUs
- No additional cost
- Best-effort basis (not guaranteed)

**Use Cases:**
- Occasional spikes
- Batch processing
- Data migration
- Temporary load

### Rate Limiting (429 Errors)

#### Causes
- Exceeded provisioned RU/s
- Hot partition (>10K RU/s)
- Burst capacity exhausted

#### Handling 429s
```csharp
var response = await container.CreateItemAsync(item);

if (response.StatusCode == System.Net.HttpStatusCode.TooManyRequests)
{
    // Wait for retry-after duration
    await Task.Delay(response.Headers.RetryAfter);
    // Retry operation
}

// Or use SDK's built-in retry
var options = new CosmosClientOptions
{
    MaxRetryAttemptsOnRateLimitedRequests = 9,
    MaxRetryWaitTimeOnRateLimitedRequests = TimeSpan.FromSeconds(30)
};
```

---

## Query Optimization

### Query Performance Fundamentals

#### Point Reads (Best Performance)
```csharp
// 1 RU for 1 KB item
var response = await container.ReadItemAsync<Item>(
    id: "item-123",
    partitionKey: new PartitionKey("partition-value")
);
```

**Characteristics:**
- Lowest RU cost
- Predictable latency (<10ms)
- Requires ID and partition key

#### Single-Partition Queries
```sql
-- Executes within one partition
SELECT * FROM c
WHERE c.partitionKey = 'partition-value'
AND c.status = 'active'
```

**Characteristics:**
- Lower RU cost than cross-partition
- Faster execution
- Scales linearly

#### Cross-Partition Queries
```sql
-- Fan-out across all partitions
SELECT * FROM c
WHERE c.status = 'active'
```

**Characteristics:**
- Higher RU cost
- Higher latency
- Parallel execution
- Use sparingly

### Query Optimization Techniques

#### 1. Use Partition Key in Filters
```sql
-- Good: Single partition
SELECT * FROM c
WHERE c.partitionKey = 'value'
AND c.status = 'active'

-- Avoid: Cross-partition
SELECT * FROM c
WHERE c.status = 'active'
```

#### 2. Use Composite Indexes
```javascript
// Indexing policy
{
  "compositeIndexes": [
    [
      {"path": "/category", "order": "ascending"},
      {"path": "/price", "order": "descending"}
    ]
  ]
}
```

```sql
-- Efficient with composite index
SELECT * FROM c
WHERE c.category = 'electronics'
ORDER BY c.price DESC
```

#### 3. Avoid SELECT *
```sql
-- Bad: Returns all properties
SELECT * FROM c WHERE c.partitionKey = 'value'

-- Good: Select only needed properties
SELECT c.id, c.name, c.price
FROM c
WHERE c.partitionKey = 'value'
```

#### 4. Use EXISTS for Filtering
```sql
-- Check for array element existence
SELECT * FROM c
WHERE EXISTS(
    SELECT VALUE tag
    FROM tag IN c.tags
    WHERE tag = 'featured'
)
```

#### 5. Limit Results
```sql
-- Use TOP for pagination
SELECT TOP 100 * FROM c
WHERE c.category = 'electronics'
ORDER BY c.price

-- Or OFFSET/LIMIT
SELECT * FROM c
ORDER BY c.price
OFFSET 100 LIMIT 100
```

### Pagination

#### Continuation Tokens
```csharp
var query = container.GetItemQueryIterator<Item>(
    "SELECT * FROM c WHERE c.category = 'electronics'",
    requestOptions: new QueryRequestOptions
    {
        MaxItemCount = 100,
        PartitionKey = new PartitionKey("electronics")
    }
);

while (query.HasMoreResults)
{
    var response = await query.ReadNextAsync();
    foreach (var item in response)
    {
        // Process item
    }
    // response.ContinuationToken for next page
}
```

**Best Practices:**
- Store continuation token client-side
- Set appropriate MaxItemCount
- Don't rely on specific page sizes

### Query Metrics
```csharp
var query = container.GetItemQueryIterator<Item>(
    "SELECT * FROM c WHERE c.category = 'electronics'",
    requestOptions: new QueryRequestOptions
    {
        PopulateIndexMetrics = true
    }
);

var response = await query.ReadNextAsync();
Console.WriteLine($"RU Charge: {response.RequestCharge}");
Console.WriteLine($"Index Metrics: {response.IndexMetrics}");
```

---

## Indexing Strategies

### Index Types

#### Range Index (Default)
- Supports equality, range, ORDER BY
- All paths indexed by default
- Efficient for most queries

#### Spatial Index
```javascript
{
  "spatialIndexes": [
    {
      "path": "/location/*",
      "types": ["Point", "Polygon"]
    }
  ]
}
```

**Query:**
```sql
SELECT * FROM c
WHERE ST_DISTANCE(c.location, {
    "type": "Point",
    "coordinates": [-122.33, 47.61]
}) < 1000
```

#### Composite Index
```javascript
{
  "compositeIndexes": [
    [
      {"path": "/category", "order": "ascending"},
      {"path": "/price", "order": "descending"},
      {"path": "/rating", "order": "descending"}
    ]
  ]
}
```

**Benefits:**
- Optimizes ORDER BY on multiple properties
- Reduces RU cost significantly
- Required for ORDER BY multiple properties

### Indexing Policy

#### Default Policy
```javascript
{
  "indexingMode": "consistent",
  "automatic": true,
  "includedPaths": [
    {"path": "/*"}
  ],
  "excludedPaths": [
    {"path": "/\"_etag\"/?"}
  ]
}
```

#### Custom Policy Example
```javascript
{
  "indexingMode": "consistent",
  "automatic": true,
  "includedPaths": [
    {"path": "/category/?"},
    {"path": "/price/?"},
    {"path": "/tags/[]/?"}
  ],
  "excludedPaths": [
    {"path": "/*"},
    {"path": "/\"_etag\"/?"}
  ],
  "compositeIndexes": [
    [
      {"path": "/category", "order": "ascending"},
      {"path": "/price", "order": "descending"}
    ]
  ],
  "spatialIndexes": [
    {"path": "/location/*", "types": ["Point"]}
  ]
}
```

### Indexing Modes

#### Consistent (Default)
- Index updated synchronously
- Queries always consistent
- Write cost includes indexing

#### None
- No indexing
- Lowest write cost
- Only point reads possible

### Indexing Best Practices

**1. Exclude Unused Paths**
```javascript
// Reduce write RU cost
{
  "excludedPaths": [
    {"path": "/largeBlob/*"},
    {"path": "/metadata/*"}
  ]
}
```

**2. Include Only Queried Paths**
```javascript
// Selective inclusion
{
  "includedPaths": [
    {"path": "/id/?"},
    {"path": "/name/?"},
    {"path": "/category/?"}
  ],
  "excludedPaths": [
    {"path": "/*"}
  ]
}
```

**3. Use Composite Indexes for ORDER BY**
Required for ORDER BY multiple properties.

**4. Use Wildcards for Arrays**
```javascript
{"path": "/tags/[]/?"}  // Index all array elements
```

**5. Monitor Index Usage**
- Check query metrics
- Use Index Metrics in response
- Adjust policy based on patterns

---

## Consistency Levels

### Five Consistency Levels

#### 1. Strong
**Guarantees:**
- Linearizability
- Reads always return latest write
- Global strong consistency

**Cost:**
- Highest read RU (2x)
- Higher latency
- Reduced availability during outages

**Use Cases:**
- Financial transactions
- Inventory management
- Auction systems

#### 2. Bounded Staleness
**Guarantees:**
- Reads lag behind writes by:
  - K versions (e.g., 100,000 operations), OR
  - T time interval (e.g., 5 minutes)
- Consistent prefix (reads in order)

**Cost:**
- Higher read RU (2x)
- Lower latency than Strong

**Use Cases:**
- Score tracking
- Status tracking
- Workflows with acceptable lag

#### 3. Session (Default)
**Guarantees:**
- Read your own writes
- Monotonic reads
- Consistent within client session

**Cost:**
- Balanced RU cost (1x)
- Good performance

**Use Cases:**
- Most applications (default choice)
- Shopping carts
- User profiles
- Social media

#### 4. Consistent Prefix
**Guarantees:**
- Reads never see out-of-order writes
- No gaps in write sequence
- May see stale data

**Cost:**
- Lower read RU (1x)

**Use Cases:**
- Social media feeds
- Status updates
- Non-critical data

#### 5. Eventual
**Guarantees:**
- Lowest consistency
- Reads may see any order
- Eventually consistent

**Cost:**
- Lowest read RU (0.5x)
- Highest availability

**Use Cases:**
- Product reviews
- Comments
- Non-critical aggregates

### Setting Consistency

#### Account-Level Default
```csharp
// Set via Portal or CLI
// Default for all operations
```

#### Request-Level Override
```csharp
var response = await container.ReadItemAsync<Item>(
    id: "item-123",
    partitionKey: new PartitionKey("value"),
    requestOptions: new ItemRequestOptions
    {
        ConsistencyLevel = ConsistencyLevel.Eventual
    }
);
```

**Note:** Can only relax, not strengthen consistency

### Consistency Decision Matrix

| Requirement | Recommended Level |
|-------------|------------------|
| Financial accuracy | Strong |
| Real-time inventory | Strong |
| User profile (own edits) | Session |
| Product catalog | Session |
| Leaderboard (acceptable lag) | Bounded Staleness |
| Social media feed | Consistent Prefix |
| Product reviews | Eventual |

---

## Change Feed

### Overview
Continuous log of all changes to container.

**Characteristics:**
- Immutable, append-only
- Ordered within partition
- Persisted (retention: varies)
- Available for all items and operations

### Change Feed Modes

#### Latest Version Mode (Default)
- Returns final state of item
- Inserts and updates
- Deletes not included (use TTL or soft delete)

#### All Versions and Deletes Mode (Preview)
- Returns all intermediate states
- Includes deletes
- Full audit trail

### Change Feed Processor

#### Basic Setup
```csharp
Container monitoredContainer = client
    .GetDatabase("sourceDb")
    .GetContainer("sourceContainer");

Container leaseContainer = client
    .GetDatabase("leaseDb")
    .GetContainer("leases");

ChangeFeedProcessor processor = monitoredContainer
    .GetChangeFeedProcessorBuilder<Item>("myProcessor", HandleChangesAsync)
    .WithInstanceName("consumerInstance1")
    .WithLeaseContainer(leaseContainer)
    .Build();

await processor.StartAsync();

async Task HandleChangesAsync(
    IReadOnlyCollection<Item> changes,
    CancellationToken cancellationToken)
{
    foreach (var item in changes)
    {
        // Process each change
        Console.WriteLine($"Changed: {item.Id}");
    }
}
```

#### Advanced Configuration
```csharp
var processor = container
    .GetChangeFeedProcessorBuilder<Item>("myProcessor", HandleChangesAsync)
    .WithInstanceName("instance1")
    .WithLeaseContainer(leaseContainer)
    .WithStartTime(DateTime.UtcNow.AddHours(-1)) // Start from 1 hour ago
    .WithPollInterval(TimeSpan.FromSeconds(5))
    .WithMaxItems(100)
    .WithLeaseConfiguration(
        acquireInterval: TimeSpan.FromSeconds(13),
        expirationInterval: TimeSpan.FromSeconds(60),
        renewInterval: TimeSpan.FromSeconds(17))
    .Build();
```

### Change Feed Use Cases

#### 1. Event-Driven Architecture
```csharp
async Task HandleChangesAsync(IReadOnlyCollection<Order> changes)
{
    foreach (var order in changes)
    {
        if (order.Status == "Placed")
        {
            await SendOrderConfirmationEmail(order);
            await UpdateInventory(order);
            await NotifyWarehouse(order);
        }
    }
}
```

#### 2. Data Replication
```csharp
async Task HandleChangesAsync(IReadOnlyCollection<Item> changes)
{
    // Replicate to another container/database
    foreach (var item in changes)
    {
        await targetContainer.UpsertItemAsync(item);
    }
}
```

#### 3. Materialized Views
```csharp
async Task HandleChangesAsync(IReadOnlyCollection<Order> changes)
{
    // Maintain aggregated view
    foreach (var order in changes)
    {
        var summary = await GetCustomerSummary(order.CustomerId);
        summary.TotalOrders++;
        summary.TotalSpent += order.Amount;
        await summaryContainer.UpsertItemAsync(summary);
    }
}
```

#### 4. Real-Time Analytics
```csharp
async Task HandleChangesAsync(IReadOnlyCollection<Event> changes)
{
    // Stream to analytics system
    await analyticsClient.SendBatch(changes);
}
```

### Change Feed vs Triggers

| Aspect | Change Feed | Triggers |
|--------|-------------|----------|
| Execution | External processor | Server-side |
| Scalability | Highly scalable | Limited |
| Reliability | Durable, resumable | Atomic with operation |
| Use Case | Async processing | Synchronous validation |

---

## Monitoring and Diagnostics

### Azure Monitor Integration

#### Metrics
**Key Metrics:**
- Total Request Units
- Total Requests
- Storage Used
- Availability
- Normalized RU Consumption (per partition)
- Physical Partition Throughput
- Physical Partition Storage

**Access:**
- Azure Portal → Metrics
- Azure Monitor API
- Application Insights

#### Diagnostic Logs
```powershell
# Enable diagnostic settings
az monitor diagnostic-settings create \
    --name CosmosDBDiagnostics \
    --resource $resourceId \
    --logs '[{"category":"DataPlaneRequests","enabled":true}]' \
    --metrics '[{"category":"Requests","enabled":true}]' \
    --workspace $workspaceId
```

**Log Categories:**
- DataPlaneRequests
- QueryRuntimeStatistics
- PartitionKeyStatistics
- ControlPlaneRequests
- MongoRequests (if using MongoDB API)

### Performance Monitoring

#### Request Diagnostics
```csharp
var response = await container.ReadItemAsync<Item>(
    id: "item-123",
    partitionKey: new PartitionKey("value")
);

Console.WriteLine(response.Diagnostics);
// Includes: request timeline, regions contacted, retry info
```

#### Query Statistics
```csharp
var queryable = container.GetItemLinqQueryable<Item>(
    requestOptions: new QueryRequestOptions
    {
        PopulateIndexMetrics = true
    }
);

var iterator = queryable.ToFeedIterator();
var response = await iterator.ReadNextAsync();

Console.WriteLine($"RU Charge: {response.RequestCharge}");
Console.WriteLine($"Index Metrics: {response.IndexMetrics}");
```

### Hot Partition Detection
```kusto
// Log Analytics query
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.DOCUMENTDB"
| where Category == "PartitionKeyRUConsumption"
| summarize TotalRU = sum(todouble(requestCharge_s)) by partitionKey_s
| order by TotalRU desc
```

### Alerts
```csharp
// Example: Alert on high RU consumption
{
  "alertName": "HighRUConsumption",
  "metricName": "TotalRequestUnits",
  "threshold": 1000000,
  "timeAggregation": "Total",
  "windowSize": "PT5M",
  "severity": 2
}
```

---

## Backup and Disaster Recovery

### Backup Modes

#### Continuous Backup
**Features:**
- Point-in-time restore (PITR)
- Restore to any second in past 30 days
- Self-service restore via Portal/CLI/PS
- Accidental delete/update recovery

**Limitations:**
- Additional cost (~1/5 of storage cost)
- Not available for serverless
- Longer restore times

**Enable:**
```powershell
az cosmosdb create \
    --name myaccount \
    --resource-group myRG \
    --backup-policy-type Continuous
```

**Restore:**
```powershell
az cosmosdb sql database restore \
    --account-name myaccount \
    --resource-group myRG \
    --name mydb \
    --restore-timestamp "2024-01-15T10:00:00Z"
```

#### Periodic Backup (Default)
**Features:**
- Automatic backups every 4 hours (default)
- Retention: 8 hours (configurable to 720 hours/30 days)
- Restore requires support ticket
- Geo-redundant storage

**Configuration:**
```powershell
az cosmosdb update \
    --name myaccount \
    --resource-group myRG \
    --backup-interval 240 \
    --backup-retention 48
```

### Multi-Region Deployment

#### Write Regions

**Single-Region Writes:**
- One write region
- Multiple read regions
- Automatic failover to read region

**Multi-Region Writes:**
- Multiple write regions
- Lower write latency globally
- Conflict resolution policies

**Enable Multi-Region:**
```csharp
await client.GetDatabase("mydb")
    .SetThroughputAsync(
        ThroughputProperties.CreateManualThroughput(10000),
        new RequestOptions { EnableMultipleWriteLocations = true }
    );
```

#### Conflict Resolution

**Last Write Wins (Default):**
```javascript
{
  "conflictResolutionPolicy": {
    "mode": "LastWriterWins",
    "conflictResolutionPath": "/_ts"
  }
}
```

**Custom (Stored Procedure):**
```javascript
{
  "conflictResolutionPolicy": {
    "mode": "Custom",
    "conflictResolutionProcedure": "dbs/mydb/colls/mycoll/sprocs/resolver"
  }
}
```

**Manual:**
```javascript
{
  "conflictResolutionPolicy": {
    "mode": "Custom"
  }
}
// Read from conflicts feed and resolve manually
```

### Disaster Recovery

#### Automatic Failover
```powershell
# Enable automatic failover
az cosmosdb update \
    --name myaccount \
    --resource-group myRG \
    --enable-automatic-failover true

# Set failover priorities
az cosmosdb failover-priority-change \
    --name myaccount \
    --resource-group myRG \
    --failover-policies "eastus=0 westus=1"
```

#### Manual Failover
```powershell
az cosmosdb failover-priority-change \
    --name myaccount \
    --resource-group myRG \
    --failover-policies "westus=0 eastus=1"
```

### SLA and Availability

| Configuration | Availability SLA |
|---------------|-----------------|
| Single region | 99.99% |
| Multiple read regions | 99.99% (read) |
| Single write region | 99.99% (write) |
| Multiple write regions | 99.999% (write) |
| Availability Zones | 99.995% (single region) |

---

## Exam Tips

### Key Concepts

**Request Units:**
- 1 RU = 1 KB point read
- Writes cost ~5-6 RUs per KB
- Strong consistency costs 2x reads
- Cross-partition queries cost more

**Throughput:**
- Manual: Fixed RU/s
- Autoscale: 0.1x to max
- Serverless: Pay per RU
- Database vs container level

**Consistency:**
- Strong: Linearizable, highest cost
- Session: Default, read-your-writes
- Eventual: Lowest cost, eventual consistency

**Indexing:**
- Default: All paths indexed
- Composite: Required for multi-property ORDER BY
- Exclude unused paths to reduce write cost

**Change Feed:**
- Ordered within partition
- Latest version mode (default)
- Use for events, replication, views
- Requires lease container

**Backup:**
- Continuous: PITR, self-service restore
- Periodic: Every 4 hours, support ticket
- Multi-region for DR

### Common Scenarios

**High RU Cost:**
1. Check query patterns (cross-partition?)
2. Review indexing policy (over-indexing?)
3. Optimize queries (SELECT *, ORDER BY?)
4. Consider caching

**Slow Queries:**
1. Use point reads when possible
2. Include partition key in WHERE
3. Add composite indexes
4. Avoid cross-partition queries

**Hot Partition:**
1. Review partition key choice
2. Use synthetic keys
3. Increase throughput
4. Monitor partition metrics

**Multi-Region Setup:**
- Single write: Lower cost, automatic failover
- Multi-write: Lower latency, conflict resolution
- Session consistency: Most common
- Automatic failover: Enable for HA

### Decision Matrix

**Throughput Model:**
- Predictable load → Manual provisioned
- Variable load → Autoscale
- Small/dev → Serverless

**Consistency Level:**
- Financial/inventory → Strong
- User profiles → Session (default)
- Product reviews → Eventual

**Backup Mode:**
- Accidental deletes concern → Continuous
- Cost-sensitive → Periodic
- Compliance requirements → Check retention

### Quick Reference

**RU Guidelines:**
- Point read: 1 RU/KB
- Write: ~5 RU/KB
- Query: 2.3+ RU minimum
- Strong consistency: 2x reads

**Throughput Scaling:**
- Manual: Scale up/down anytime
- Autoscale: Instant scale up, gradual down
- Per partition: Max 10K RU/s

**Indexing Best Practices:**
- Exclude large, unused properties
- Use composite for ORDER BY
- Include only queried paths
- Monitor with index metrics

**Change Feed:**
- Lease container required
- Horizontal scaling supported
- At-least-once delivery
- Ordered per partition

### Study Focus

1. **Understand RU consumption** patterns
2. **Master query optimization** techniques
3. **Know indexing strategies** and impact
4. **Understand consistency** tradeoffs
5. **Learn change feed** use cases
6. **Practice monitoring** and diagnostics
7. **Know backup** and DR options

### Final Checklist

- [ ] RU consumption factors
- [ ] Throughput models (manual, autoscale, serverless)
- [ ] Query optimization techniques
- [ ] Indexing policy configuration
- [ ] Consistency level characteristics
- [ ] Change feed patterns
- [ ] Monitoring and diagnostics
- [ ] Backup modes and restore
- [ ] Multi-region configuration
- [ ] Performance troubleshooting
