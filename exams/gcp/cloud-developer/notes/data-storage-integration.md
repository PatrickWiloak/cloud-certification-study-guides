# Data Storage and Integration - GCP Professional Cloud Developer

## Overview

Integrating applications with GCP data services, messaging systems, and implementing data access patterns for cloud-native applications.

## Key Storage Services

### Cloud SQL
**Use Cases**: Relational data, transactional workloads, legacy app migration
**Connection Methods**:
- Cloud SQL Proxy (recommended for security)
- Public IP with authorized networks
- Private IP within VPC
- Unix socket for App Engine

**Best Practices**:
```python
import sqlalchemy

# Connection pooling
pool = sqlalchemy.create_engine(
    "postgresql+pg8000://user:password@/dbname",
    pool_size=5,
    max_overflow=2,
    pool_timeout=30,
    pool_recycle=1800
)
```

### Firestore
**Native Mode** (recommended):
- Document database
- Real-time synchronization
- Offline support
- Auto-scaling
- Mobile/web SDKs

**Client Libraries**:
```python
from google.cloud import firestore

db = firestore.Client()

# Add document
doc_ref = db.collection('users').document('user123')
doc_ref.set({'name': 'John', 'age': 30})

# Query
users = db.collection('users').where('age', '>=', 18).stream()
for user in users:
    print(user.to_dict())

# Real-time listener
def on_snapshot(doc_snapshot, changes, read_time):
    for doc in doc_snapshot:
        print(f'Received document: {doc.id}')

doc_ref.on_snapshot(on_snapshot)
```

### Cloud Storage
**Use Cases**: Object storage, static files, user uploads, data lakes
**Access Methods**:
- Client libraries
- Signed URLs for temporary access
- Direct API calls
- gsutil CLI

**Implementation**:
```python
from google.cloud import storage

client = storage.Client()
bucket = client.bucket('my-bucket')

# Upload file
blob = bucket.blob('path/to/file.txt')
blob.upload_from_filename('local-file.txt')

# Generate signed URL
url = blob.generate_signed_url(expiration=3600, method='GET')

# Stream upload
with blob.open('w') as f:
    f.write('Large file content')
```

### Memorystore (Redis/Memcached)
**Use Cases**: Session storage, caching, real-time analytics
**Connection**: Within VPC, serverless VPC connector for Cloud Run/Functions

**Caching Pattern**:
```python
import redis

r = redis.Redis(host='10.0.0.3', port=6379)

def get_user(user_id):
    # Check cache
    cached = r.get(f'user:{user_id}')
    if cached:
        return json.loads(cached)

    # Fetch from database
    user = fetch_from_db(user_id)

    # Store in cache (1 hour TTL)
    r.setex(f'user:{user_id}', 3600, json.dumps(user))
    return user
```

## Integration Patterns

### Pub/Sub Integration
**Asynchronous Processing**:
```python
from google.cloud import pubsub_v1

# Publisher
publisher = pubsub_v1.PublisherClient()
topic_path = publisher.topic_path('project', 'topic-name')

message = b'Message payload'
future = publisher.publish(topic_path, message, attribute='value')
message_id = future.result()

# Subscriber
subscriber = pubsub_v1.SubscriberClient()
subscription_path = subscriber.subscription_path('project', 'subscription-name')

def callback(message):
    print(f'Received message: {message.data}')
    message.ack()

streaming_pull_future = subscriber.subscribe(subscription_path, callback=callback)
```

**Use Cases**:
- Decoupling services
- Event-driven architectures
- Fan-out messaging
- Buffering traffic spikes

### Cloud Tasks
**Task Queue Management**:
```python
from google.cloud import tasks_v2

client = tasks_v2.CloudTasksClient()
parent = client.queue_path('project', 'location', 'queue-name')

task = {
    'http_request': {
        'http_method': tasks_v2.HttpMethod.POST,
        'url': 'https://myapp.com/process',
        'headers': {'Content-Type': 'application/json'},
        'body': json.dumps({'key': 'value'}).encode()
    }
}

# Schedule task
response = client.create_task(request={'parent': parent, 'task': task})
```

**Use Cases**:
- Deferred processing
- Rate-limited operations
- Reliable task execution
- Scheduled jobs

## Data Access Patterns

### Repository Pattern
```python
class UserRepository:
    def __init__(self, db):
        self.db = db

    def get_user(self, user_id):
        return self.db.collection('users').document(user_id).get()

    def create_user(self, user_data):
        doc_ref = self.db.collection('users').document()
        doc_ref.set(user_data)
        return doc_ref.id

    def update_user(self, user_id, updates):
        doc_ref = self.db.collection('users').document(user_id)
        doc_ref.update(updates)
```

### Caching Layer Pattern
```python
class CachedUserService:
    def __init__(self, repository, cache):
        self.repo = repository
        self.cache = cache

    def get_user(self, user_id):
        # Try cache first
        cached = self.cache.get(f'user:{user_id}')
        if cached:
            return json.loads(cached)

        # Fetch from repository
        user = self.repo.get_user(user_id)

        # Update cache
        self.cache.setex(f'user:{user_id}', 3600, json.dumps(user))
        return user
```

### Event Sourcing Pattern
```python
class OrderEventStore:
    def __init__(self, db):
        self.db = db

    def append_event(self, order_id, event_type, event_data):
        event = {
            'event_type': event_type,
            'event_data': event_data,
            'timestamp': datetime.utcnow()
        }
        self.db.collection('orders').document(order_id) \
            .collection('events').add(event)

    def get_order_state(self, order_id):
        events = self.db.collection('orders').document(order_id) \
            .collection('events').order_by('timestamp').stream()

        state = {}
        for event in events:
            state = self.apply_event(state, event.to_dict())
        return state
```

## Best Practices

### Connection Management
1. Use connection pooling
2. Implement retry logic with exponential backoff
3. Handle transient failures
4. Close connections properly
5. Use Cloud SQL Proxy for security
6. Configure appropriate timeouts
7. Monitor connection metrics

### Data Consistency
1. Use transactions where needed
2. Implement idempotency
3. Handle eventual consistency
4. Use optimistic locking
5. Implement conflict resolution
6. Validate data integrity
7. Use appropriate isolation levels

### Performance Optimization
1. Cache frequently accessed data
2. Use batch operations
3. Implement pagination
4. Optimize database indexes
5. Use denormalization when appropriate
6. Implement data sharding
7. Monitor query performance
8. Use CDN for static assets

### Security
1. Use least privilege for service accounts
2. Encrypt sensitive data
3. Use VPC for private connectivity
4. Implement rate limiting
5. Validate and sanitize inputs
6. Use signed URLs for temporary access
7. Enable audit logging
8. Regular security reviews

## Common Scenarios

**Scenario**: User authentication and session management
**Solution**: Firebase Auth for authentication, Firestore for user profiles, Memorystore for sessions

**Scenario**: File upload processing
**Solution**: Cloud Storage trigger → Cloud Functions → Process file → Store metadata in Firestore

**Scenario**: Async job processing
**Solution**: HTTP endpoint → Pub/Sub publish → Cloud Run subscriber → Process job → Update Firestore

**Scenario**: E-commerce cart management
**Solution**: Memorystore for active carts, Firestore for persistence, Cloud SQL for orders

## Study Tips

1. **Practice with client libraries**: Write code to interact with each service
2. **Understand connection patterns**: Cloud SQL Proxy, VPC connectors
3. **Async processing**: Pub/Sub and Cloud Tasks use cases
4. **Caching strategies**: When and how to implement caching
5. **Transaction handling**: ACID properties in Firestore and Cloud SQL
6. **Performance optimization**: Connection pooling, batching, indexing
7. **Security best practices**: Authentication, encryption, access control

## Key Code Patterns

```python
# Firestore transaction
@firestore.transactional
def update_balance(transaction, account_ref, amount):
    snapshot = account_ref.get(transaction=transaction)
    balance = snapshot.get('balance')
    transaction.update(account_ref, {'balance': balance + amount})

# Cloud SQL with context manager
with pool.connect() as conn:
    result = conn.execute("SELECT * FROM users WHERE id = ?", (user_id,))
    user = result.fetchone()

# Pub/Sub with error handling
def publish_message(message):
    try:
        future = publisher.publish(topic_path, message.encode())
        return future.result(timeout=30)
    except Exception as e:
        logging.error(f'Failed to publish: {e}')
        raise

# Exponential backoff retry
from google.api_core import retry

@retry.Retry(predicate=retry.if_transient_error)
def query_with_retry():
    return client.query(sql).result()
```

## Additional Resources

- [Cloud SQL Best Practices](https://cloud.google.com/sql/docs/postgres/best-practices)
- [Firestore Data Model](https://firebase.google.com/docs/firestore/data-model)
- [Pub/Sub Documentation](https://cloud.google.com/pubsub/docs)
- [Memorystore Documentation](https://cloud.google.com/memorystore)
