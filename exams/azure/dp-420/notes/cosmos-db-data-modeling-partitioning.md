# Cosmos DB Data Modeling and Partitioning

## Table of Contents
1. [Data Modeling Fundamentals](#data-modeling-fundamentals)
2. [Partitioning Strategy](#partitioning-strategy)
3. [Container Design Patterns](#container-design-patterns)
4. [Hierarchical Partition Keys](#hierarchical-partition-keys)
5. [Data Distribution and Scalability](#data-distribution-and-scalability)
6. [Migration and Modeling Best Practices](#migration-and-modeling-best-practices)
7. [Exam Tips](#exam-tips)

---

## Data Modeling Fundamentals

### NoSQL vs Relational Modeling

#### Key Differences
**Relational (SQL):**
- Normalized data
- Multiple tables with relationships
- JOIN operations
- Schema enforcement
- ACID across tables

**NoSQL (Cosmos DB):**
- Denormalized data
- Document-oriented
- No JOINs across containers
- Schema-free (schema-agnostic)
- ACID within partition

### Document Structure

#### Item Properties
```json
{
  "id": "unique-identifier",
  "partitionKey": "partition-value",
  "customProperty1": "value",
  "nestedObject": {
    "property": "value"
  },
  "arrayProperty": [1, 2, 3],
  "_rid": "system-resource-id",
  "_self": "system-self-link",
  "_etag": "concurrency-token",
  "_ts": 1234567890
}
```

**System Properties:**
- **id** - Unique within partition (required)
- **_rid** - Resource ID (internal)
- **_etag** - Concurrency control
- **_ts** - Last modified timestamp
- **_self** - Resource URI

#### Item Size Limits
- Maximum item size: 2 MB
- Includes all properties and metadata
- Consider splitting large documents
- Use references for large data

### Modeling Approaches

#### Embedding (Denormalization)
Store related data within same document.

**Advantages:**
- Single read operation
- Better performance
- Strong consistency
- Simpler queries

**Disadvantages:**
- Data duplication
- Larger item sizes
- Update complexity
- Unbounded growth risk

**Example - E-commerce Order:**
```json
{
  "id": "order-123",
  "customerId": "customer-456",
  "orderDate": "2024-01-15",
  "status": "shipped",
  "customer": {
    "name": "John Doe",
    "email": "john@example.com",
    "shippingAddress": {
      "street": "123 Main St",
      "city": "Seattle",
      "state": "WA",
      "zip": "98101"
    }
  },
  "items": [
    {
      "productId": "prod-789",
      "name": "Widget",
      "quantity": 2,
      "price": 29.99
    },
    {
      "productId": "prod-101",
      "name": "Gadget",
      "quantity": 1,
      "price": 49.99
    }
  ],
  "totalAmount": 109.97,
  "partitionKey": "customer-456"
}
```

**When to Embed:**
- One-to-few relationships
- Data read together
- Infrequent updates
- Bounded relationship (won't grow indefinitely)

#### Referencing (Normalization)
Store related data in separate documents.

**Advantages:**
- No data duplication
- Smaller item sizes
- Easier updates
- Handles unbounded relationships

**Disadvantages:**
- Multiple read operations
- Higher latency
- More RU consumption
- Application-side joins

**Example - Separate Documents:**
```json
// Customer document
{
  "id": "customer-456",
  "name": "John Doe",
  "email": "john@example.com",
  "partitionKey": "customer-456"
}

// Order document
{
  "id": "order-123",
  "customerId": "customer-456",
  "orderDate": "2024-01-15",
  "totalAmount": 109.97,
  "partitionKey": "customer-456"
}

// Order items (separate documents)
{
  "id": "orderitem-001",
  "orderId": "order-123",
  "productId": "prod-789",
  "quantity": 2,
  "partitionKey": "order-123"
}
```

**When to Reference:**
- One-to-many with many items
- Frequently updated reference data
- Data read separately
- Unbounded relationships

### Hybrid Approach
Combine embedding and referencing.

**Example:**
```json
{
  "id": "order-123",
  "customerId": "customer-456",
  "orderDate": "2024-01-15",
  // Embed frequently accessed customer data
  "customerName": "John Doe",
  "customerEmail": "john@example.com",
  // Embed order items (bounded)
  "items": [
    {"productId": "prod-789", "quantity": 2, "price": 29.99}
  ],
  // Reference for full customer details
  "customerRef": "/customers/customer-456",
  "partitionKey": "customer-456"
}
```

### Handling Different Entity Types

#### Single Container with Type Discriminator
```json
// Customer
{
  "id": "customer-456",
  "type": "customer",
  "name": "John Doe",
  "partitionKey": "customer-456"
}

// Order
{
  "id": "order-123",
  "type": "order",
  "customerId": "customer-456",
  "orderDate": "2024-01-15",
  "partitionKey": "customer-456"
}
```

**Query with Filter:**
```sql
SELECT * FROM c
WHERE c.partitionKey = 'customer-456'
AND c.type = 'order'
```

#### Separate Containers
- Customer container
- Order container
- Product container

**Considerations:**
- No cross-container transactions
- No cross-container queries
- Separate throughput allocation
- Simpler data model per container

---

## Partitioning Strategy

### Partition Key Fundamentals

#### What is a Partition Key?
- Property that determines data distribution
- Routes data to physical partitions
- Cannot be changed after item creation
- Required for every document

#### Partition Key Path
```javascript
// Create container with partition key
{
  "id": "myContainer",
  "partitionKey": {
    "paths": ["/customerId"],
    "kind": "Hash"
  }
}
```

### Physical vs Logical Partitions

#### Logical Partition
- All items with same partition key value
- Maximum 20 GB size
- Unlimited logical partitions
- Single physical partition or distributed

#### Physical Partition
- Managed by Cosmos DB
- 10 GB to 50 GB storage
- Up to 10,000 RU/s throughput
- Automatic splitting and merging
- Transparent to application

### Choosing a Partition Key

#### Key Criteria

**1. Cardinality (Number of Distinct Values)**
- High cardinality preferred
- More values = better distribution
- Low cardinality = hot partitions

**Good:**
- User ID (millions of users)
- Order ID (millions of orders)
- Device ID (millions of devices)

**Poor:**
- Status (few values: active, inactive)
- Category (limited categories)
- Boolean flags

**2. Query Patterns**
- Partition key in WHERE clause
- Avoid cross-partition queries
- Design for common query patterns

**3. Write Distribution**
- Even write distribution
- Avoid hot partitions
- Consider time-based patterns

**4. Storage Distribution**
- Even data distribution
- Keep logical partition < 20 GB
- Monitor partition sizes

### Partition Key Anti-Patterns

#### Anti-Pattern 1: Low Cardinality
```json
// BAD: Status as partition key
{
  "id": "order-123",
  "status": "active", // Only a few values
  "partitionKey": "active"
}
```

**Problem:** Hot partitions, uneven distribution

#### Anti-Pattern 2: Time-Based (Current Time)
```json
// BAD: Current date as partition key
{
  "id": "event-123",
  "eventDate": "2024-01-15",
  "partitionKey": "2024-01-15"
}
```

**Problem:** All writes go to today's partition (hot partition)

**Better:** Combine with another property
```json
{
  "id": "event-123",
  "deviceId": "device-456",
  "eventDate": "2024-01-15",
  "partitionKey": "device-456" // or "device-456-2024-01-15"
}
```

#### Anti-Pattern 3: Unbounded Growth
```json
// BAD: Single partition key for all versions
{
  "id": "doc-v1",
  "documentId": "doc-123",
  "version": 1,
  "partitionKey": "doc-123" // Will exceed 20 GB
}
```

**Problem:** Logical partition exceeds 20 GB limit

### Synthetic Partition Keys

#### Concatenated Keys
Combine multiple properties for better distribution.

```json
{
  "id": "order-123",
  "customerId": "customer-456",
  "region": "west",
  "partitionKey": "west-customer-456"
}
```

#### Hashed Keys
Use hash of property for distribution.

```json
{
  "id": "user-123",
  "userId": "user-123",
  "partitionKey": "hash-7" // Hash(userId) % 10
}
```

**Benefits:**
- Even distribution
- Predictable routing

**Drawbacks:**
- More cross-partition queries
- Complex query patterns

#### Suffix/Prefix Keys
```json
{
  "id": "order-123",
  "customerId": "customer-456",
  "orderDate": "2024-01-15",
  "partitionKey": "customer-456-2024-01" // Customer + Month
}
```

### Multi-Tenant Scenarios

#### Tenant Per Partition
```json
{
  "id": "doc-123",
  "tenantId": "tenant-456",
  "data": "...",
  "partitionKey": "tenant-456"
}
```

**Pros:**
- Isolation per tenant
- Easy to query tenant data
- RLS at partition level

**Cons:**
- Limited to 20 GB per tenant
- Uneven tenant sizes
- Hot partitions for large tenants

#### Hybrid Approach
```json
{
  "id": "doc-123",
  "tenantId": "tenant-456",
  "category": "orders",
  "partitionKey": "tenant-456-orders"
}
```

---

## Container Design Patterns

### Single Container Design
Store all entity types in one container.

**Advantages:**
- Transactional consistency within partition
- Fewer containers to manage
- Cost-effective (shared throughput)
- Query related entities together

**Disadvantages:**
- Complex queries with type filters
- Indexing overhead
- Throughput shared across types

**Example:**
```json
// Customer
{
  "id": "customer-456",
  "type": "customer",
  "name": "John Doe",
  "pk": "customer-456"
}

// Order (same partition as customer)
{
  "id": "order-123",
  "type": "order",
  "customerId": "customer-456",
  "pk": "customer-456"
}

// Order Item (same partition)
{
  "id": "item-001",
  "type": "orderItem",
  "orderId": "order-123",
  "pk": "customer-456"
}
```

### Multiple Container Design
Separate containers for each entity type.

**Advantages:**
- Clear separation
- Independent indexing policies
- Independent throughput
- Simpler queries

**Disadvantages:**
- No cross-container transactions
- No cross-container queries
- Higher cost (multiple containers)
- More complex application logic

### Time Series Data Pattern

#### Hot-Warm-Cold Storage
```javascript
// Hot container (recent data, high throughput)
{
  "id": "container-hot",
  "throughput": 10000,
  "ttl": 86400 // 24 hours
}

// Warm container (older data, medium throughput)
{
  "id": "container-warm",
  "throughput": 1000,
  "ttl": 2592000 // 30 days
}

// Cold container (archive, low throughput)
{
  "id": "container-cold",
  "throughput": 400
}
```

#### Change Feed for Migration
Use Change Feed to move data between containers.

```csharp
// Pseudo-code
ChangeFeedProcessor processor = container
    .GetChangeFeedProcessorBuilder("migrator", HandleChangesAsync)
    .Build();

async Task HandleChangesAsync(IReadOnlyCollection<Item> changes)
{
    foreach (var item in changes)
    {
        if (item.Timestamp < DateTime.UtcNow.AddDays(-1))
        {
            await warmContainer.CreateItemAsync(item);
            await hotContainer.DeleteItemAsync(item.Id, item.PartitionKey);
        }
    }
}
```

### Lookup/Reference Data Pattern

**Small Reference Data:**
- Embed in documents
- Duplicate across documents
- Accept data duplication

**Large Reference Data:**
- Separate container
- Query when needed
- Cache in application

**Frequently Accessed:**
- Embed critical fields
- Reference for full details

### Event Sourcing Pattern
Store all events in Cosmos DB.

```json
{
  "id": "event-001",
  "aggregateId": "order-123",
  "eventType": "OrderCreated",
  "timestamp": "2024-01-15T10:00:00Z",
  "data": {
    "customerId": "customer-456",
    "items": [...]
  },
  "partitionKey": "order-123"
}

{
  "id": "event-002",
  "aggregateId": "order-123",
  "eventType": "OrderShipped",
  "timestamp": "2024-01-16T14:30:00Z",
  "data": {
    "trackingNumber": "ABC123"
  },
  "partitionKey": "order-123"
}
```

**Query Events:**
```sql
SELECT * FROM c
WHERE c.partitionKey = 'order-123'
ORDER BY c.timestamp
```

---

## Hierarchical Partition Keys

### Overview
Use multiple levels of partition keys (preview feature).

#### Syntax
```javascript
{
  "partitionKey": {
    "paths": ["/tenantId", "/userId", "/sessionId"],
    "kind": "MultiHash"
  }
}
```

### Benefits

**1. Better Distribution**
- More granular partitioning
- Avoid hot partitions
- Handle high cardinality

**2. Flexible Queries**
- Query at any level
- Efficient prefix queries
- Better query performance

**3. Hierarchical Access**
```sql
-- All data for tenant
SELECT * FROM c WHERE c.tenantId = 'tenant-1'

-- All data for tenant user
SELECT * FROM c WHERE c.tenantId = 'tenant-1' AND c.userId = 'user-123'

-- Specific session
SELECT * FROM c WHERE c.tenantId = 'tenant-1'
  AND c.userId = 'user-123'
  AND c.sessionId = 'session-456'
```

### Use Cases

**Multi-Tenant SaaS:**
```javascript
["/tenantId", "/userId"]
```

**IoT Scenarios:**
```javascript
["/deviceType", "/deviceId", "/date"]
```

**Time-Series Data:**
```javascript
["/category", "/year", "/month", "/day"]
```

### Limitations
- Preview feature (check current status)
- Up to 3 levels
- Cannot change after creation
- Not all SDKs support yet

---

## Data Distribution and Scalability

### Throughput Distribution

#### Container-Level Throughput
**Shared Throughput:**
- Provisioned at container level
- Distributed across physical partitions
- Each partition gets fraction of total

**Example:**
- Container throughput: 10,000 RU/s
- Physical partitions: 10
- Per partition: ~1,000 RU/s

#### Partition-Level Limits
- Max 10,000 RU/s per physical partition
- Exceeding causes rate limiting (429)
- Automatic partition splitting

### Autoscale vs Manual Throughput

#### Manual (Provisioned)
```csharp
await database.CreateContainerAsync(
    new ContainerProperties("myContainer", "/partitionKey"),
    throughput: 1000
);
```

**Characteristics:**
- Fixed RU/s
- Predictable cost
- Manual scaling
- Best for steady workloads

#### Autoscale
```csharp
await database.CreateContainerAsync(
    new ContainerProperties("myContainer", "/partitionKey"),
    ThroughputProperties.CreateAutoscaleThroughput(10000)
);
```

**Characteristics:**
- Scales 0.1x to max
- Pay for actual usage
- Automatic scaling
- Best for variable workloads

**Cost Comparison:**
- Autoscale min: 10% of max RU/s
- Charges hourly based on peak
- Manual: Always charged for provisioned

### Partition Splitting

#### Automatic Splitting
Triggers:
- Storage > 50 GB
- Throughput > 10,000 RU/s
- Approaching limits

**Process:**
1. Cosmos DB detects need
2. Creates new physical partition
3. Redistributes data
4. Transparent to application

#### Partition Merging
- Automatic when underutilized
- Optimizes resource usage
- No application impact

### Monitoring Distribution

#### Metrics to Monitor
```csharp
// Check partition key statistics
var options = new QueryRequestOptions
{
    PopulateIndexMetrics = true
};

var query = container.GetItemQueryIterator<dynamic>(
    "SELECT * FROM c",
    requestOptions: options
);

// Review metrics
while (query.HasMoreResults)
{
    var response = await query.ReadNextAsync();
    Console.WriteLine($"RU Charge: {response.RequestCharge}");
}
```

#### Portal Metrics
- Storage per partition key range
- RU consumption per partition
- Throttling rate (429 errors)
- Partition key range distribution

### Hot Partition Detection

**Signs of Hot Partition:**
- High 429 (rate limiting) errors
- Uneven RU consumption
- Specific queries slow
- Storage imbalance

**Solutions:**
1. **Re-evaluate Partition Key**
   - Choose higher cardinality
   - Use synthetic keys
   - Add randomization

2. **Increase Throughput**
   - Scale up container
   - Use autoscale
   - Temporary burst capacity

3. **Application Changes**
   - Implement caching
   - Batch operations
   - Retry logic with backoff

4. **Data Model Changes**
   - Split large partitions
   - Redistribute data
   - Use hierarchical keys

---

## Migration and Modeling Best Practices

### Migration from Relational

#### Assessment
1. **Identify Entities and Relationships**
   - Tables → Containers or documents
   - Relationships → Embedded or referenced
   - Queries → Access patterns

2. **Define Access Patterns**
   - How data is queried
   - Read vs write ratio
   - Consistency requirements

3. **Choose Partition Key**
   - Based on query patterns
   - High cardinality
   - Even distribution

#### Example Migration

**Relational Schema:**
```sql
-- Customers table
CREATE TABLE Customers (
  CustomerID INT PRIMARY KEY,
  Name VARCHAR(100),
  Email VARCHAR(100)
);

-- Orders table
CREATE TABLE Orders (
  OrderID INT PRIMARY KEY,
  CustomerID INT,
  OrderDate DATE,
  FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID)
);

-- OrderItems table
CREATE TABLE OrderItems (
  OrderItemID INT PRIMARY KEY,
  OrderID INT,
  ProductID INT,
  Quantity INT,
  FOREIGN KEY (OrderID) REFERENCES Orders(OrderID)
);
```

**Cosmos DB Model (Embedded):**
```json
{
  "id": "order-123",
  "customerId": "customer-456",
  "customer": {
    "name": "John Doe",
    "email": "john@example.com"
  },
  "orderDate": "2024-01-15",
  "items": [
    {
      "productId": "prod-789",
      "quantity": 2
    }
  ],
  "partitionKey": "customer-456"
}
```

### Data Modeling Best Practices

#### 1. Design for Queries
- Start with access patterns
- Optimize for common queries
- Accept some data duplication

#### 2. Choose Right Partition Key
- High cardinality
- Even distribution
- Frequently queried
- Avoid hot partitions

#### 3. Denormalize Strategically
- Embed frequently accessed data
- Keep item size < 2 MB
- Balance read vs write costs

#### 4. Use Appropriate Indexing
- Exclude unused properties
- Use composite indexes
- Consider spatial/array indexes

#### 5. Implement Versioning
```json
{
  "id": "doc-123-v2",
  "documentId": "doc-123",
  "version": 2,
  "data": "...",
  "partitionKey": "doc-123"
}
```

#### 6. Handle Large Documents
- Split into multiple items
- Use continuation tokens
- Reference external storage (Blob)

#### 7. Plan for Growth
- Anticipate data volume
- Keep logical partitions < 20 GB
- Monitor partition metrics

### Testing and Validation

#### Performance Testing
1. Load test with realistic data
2. Monitor RU consumption
3. Check partition distribution
4. Identify hot partitions
5. Measure query performance

#### Data Validation
1. Verify data integrity
2. Test all query patterns
3. Validate partition key choice
4. Check consistency levels
5. Test failover scenarios

---

## Exam Tips

### Key Concepts

**Data Modeling:**
- Embedding vs Referencing
- Single vs Multiple containers
- Type discriminators
- Item size limits (2 MB)

**Partition Keys:**
- Choose high cardinality
- Based on query patterns
- Cannot change after creation
- Logical partition max 20 GB

**Synthetic Keys:**
- Concatenation
- Hashing
- Prefix/Suffix
- Better distribution

**Hierarchical Keys:**
- Multiple levels (preview)
- Up to 3 levels
- Flexible queries
- Better distribution

**Scalability:**
- Physical partition: 10K RU/s max
- Logical partition: 20 GB max
- Automatic splitting/merging
- Monitor hot partitions

### Common Scenarios

**Scenario: E-commerce Application**
- Partition key: customerId
- Embed: Order items, shipping address
- Reference: Product catalog
- Single container for customer, orders, items

**Scenario: IoT Time-Series**
- Partition key: deviceId or deviceId-date
- Separate containers: hot, warm, cold
- TTL for data expiration
- Change feed for archival

**Scenario: Multi-Tenant SaaS**
- Partition key: tenantId or tenantId-category
- Hierarchical keys for large tenants
- Separate containers per tenant (if needed)
- Monitor tenant sizes

**Scenario: Hot Partition**
- Re-evaluate partition key
- Use synthetic keys
- Increase throughput
- Implement caching

### Decision Matrix

**When to Embed:**
- One-to-few relationships
- Data read together
- Infrequent updates
- Bounded growth

**When to Reference:**
- One-to-many (many items)
- Frequently updated data
- Data read separately
- Unbounded relationships

**When to Use Single Container:**
- Related entities
- Transactions needed
- Shared throughput OK
- Query related data

**When to Use Multiple Containers:**
- Unrelated entities
- Different indexing needs
- Independent throughput
- Clear separation needed

### Quick Reference

**Partition Key Characteristics:**
- High cardinality
- Even distribution
- Query optimization
- < 20 GB per value

**Item Properties:**
- id: Unique within partition
- Maximum size: 2 MB
- System properties: _rid, _etag, _ts
- Partition key: Required

**Container Types:**
- Single container: Type discriminator
- Multiple containers: Separate entities
- Time-series: Hot/warm/cold
- Reference: Lookup data

### Study Focus

1. **Understand modeling approaches** - Embedding vs referencing
2. **Master partition key selection** - Critical for performance
3. **Know scalability limits** - 20 GB logical, 10K RU/s physical
4. **Practice design scenarios** - E-commerce, IoT, multi-tenant
5. **Learn anti-patterns** - Low cardinality, unbounded growth
6. **Understand hierarchical keys** - When and how to use
7. **Know container patterns** - Single vs multiple containers

### Final Checklist

- [ ] Embedding vs referencing decisions
- [ ] Partition key selection criteria
- [ ] Synthetic partition key strategies
- [ ] Hierarchical partition keys
- [ ] Container design patterns
- [ ] Scalability limits and monitoring
- [ ] Hot partition detection and mitigation
- [ ] Migration from relational databases
- [ ] Best practices for data modeling
- [ ] Performance testing approaches
