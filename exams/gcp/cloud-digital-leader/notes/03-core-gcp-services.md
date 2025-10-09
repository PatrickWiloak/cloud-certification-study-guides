# Core Google Cloud Platform Services

## Compute Services

### Compute Engine (IaaS)
**What it is:** Virtual machines running in Google's data centers

**Key Features:**
- **Custom machine types:** Tailor CPU and memory to your needs
- **Preemptible instances:** Up to 80% cost savings for fault-tolerant workloads
- **Live migration:** Zero-downtime maintenance
- **Sustained use discounts:** Automatic discounts for long-running instances
- **Global load balancing:** Distribute traffic across regions

**Use Cases:**
- Web servers and application hosting
- Development and testing environments
- High-performance computing (HPC)
- Migration of existing applications (lift-and-shift)

**Pricing Model:** Pay per second with a 1-minute minimum

### App Engine (PaaS)
**What it is:** Platform for building and hosting web applications

**Key Features:**
- **Automatic scaling:** Scale to zero or handle traffic spikes
- **Multiple language support:** Java, Python, PHP, Go, Node.js, .NET, Ruby
- **Integrated services:** Built-in services for common application needs
- **Version management:** Deploy multiple versions and split traffic
- **No server management:** Focus on code, not infrastructure

**Use Cases:**
- Web applications and APIs
- Microservices architecture
- Rapid prototyping and development
- Applications with variable traffic patterns

**Environments:**
- **Standard:** Preconfigured runtime environments
- **Flexible:** Docker containers with more flexibility

### Google Kubernetes Engine (GKE)
**What it is:** Managed Kubernetes service for containerized applications

**Key Features:**
- **Managed Kubernetes:** Google manages the control plane
- **Auto-scaling:** Horizontal pod autoscaling and cluster autoscaling
- **Security:** Workload Identity, Binary Authorization, Pod Security Policy
- **Integrated monitoring:** Built-in logging and monitoring
- **Multi-cloud:** Anthos for hybrid and multi-cloud deployments

**Use Cases:**
- Containerized applications
- Microservices architecture
- DevOps and CI/CD pipelines
- Hybrid and multi-cloud deployments

**Node Types:**
- **Standard nodes:** Traditional VMs for containers
- **Spot nodes:** Preemptible instances for cost savings

### Cloud Functions (Serverless)
**What it is:** Event-driven serverless compute platform

**Key Features:**
- **Event-driven:** Triggered by events from GCP services
- **Auto-scaling:** Scale from zero to handle traffic
- **Pay-per-use:** Only pay for execution time
- **Multiple triggers:** HTTP, Cloud Storage, Pub/Sub, Firestore
- **Language support:** Node.js, Python, Go, Java, .NET, Ruby, PHP

**Use Cases:**
- API backends and webhooks
- Data processing and ETL
- IoT data processing
- Real-time file processing
- Serverless microservices

**Trigger Types:**
- **HTTP triggers:** Direct HTTP requests
- **Background triggers:** Cloud events (storage, database, messaging)

### Cloud Run
**What it is:** Fully managed serverless platform for containerized applications

**Key Features:**
- **Container-based:** Deploy any containerized application
- **Automatic scaling:** Scale to zero or handle traffic spikes
- **Request-based billing:** Pay only for requests and compute time
- **Portable:** Standard container runtime
- **Integrated security:** Built-in TLS and IAM integration

**Use Cases:**
- Containerized web applications
- APIs and microservices
- Batch processing jobs
- Event-driven applications

## Storage Services

### Cloud Storage (Object Storage)
**What it is:** Scalable object storage for unstructured data

**Storage Classes:**
- **Standard:** Frequently accessed data (>1x/month)
- **Nearline:** Infrequently accessed data (1x/month)
- **Coldline:** Rarely accessed data (1x/quarter)
- **Archive:** Long-term archival (1x/year)

**Key Features:**
- **Global accessibility:** Access from anywhere on the internet
- **Automatic redundancy:** Data replicated across multiple locations
- **Lifecycle management:** Automatic data class transitions
- **Security:** Encryption at rest and in transit
- **Integration:** Works with all GCP services

**Use Cases:**
- Website content and media files
- Data backup and archival
- Data lake for analytics
- Content distribution
- Disaster recovery

### Persistent Disks
**What it is:** High-performance block storage for virtual machines

**Types:**
- **Standard persistent disks:** HDD-based storage
- **SSD persistent disks:** High-performance SSD storage
- **Extreme persistent disks:** Highest performance for databases

**Key Features:**
- **Encryption:** Encrypted by default
- **Snapshots:** Point-in-time backups
- **Resizing:** Resize disks without downtime
- **Regional disks:** Replicated across zones for high availability

**Use Cases:**
- Database storage
- File systems for VMs
- High-performance computing
- Business-critical applications

### Filestore
**What it is:** Fully managed Network File System (NFS) for applications

**Key Features:**
- **High performance:** Up to 16 GB/s throughput
- **POSIX compliance:** Standard file system interface
- **Automatic backups:** Scheduled and on-demand backups
- **Integration:** Works with GKE and Compute Engine

**Use Cases:**
- Application migration requiring shared storage
- Content management systems
- Data analytics workloads
- High-performance computing

## Database Services

### Cloud SQL
**What it is:** Fully managed relational database service

**Supported Engines:**
- **MySQL:** Open-source relational database
- **PostgreSQL:** Advanced open-source database
- **SQL Server:** Microsoft's relational database

**Key Features:**
- **Automatic backups:** Point-in-time recovery
- **High availability:** Multi-zone deployments
- **Security:** Encryption, IAM integration, private IP
- **Scaling:** Read replicas and connection pooling
- **Maintenance:** Automatic updates and patches

**Use Cases:**
- Web applications
- Business applications
- Content management systems
- E-commerce platforms

### Cloud Spanner
**What it is:** Globally distributed, strongly consistent relational database

**Key Features:**
- **Global scale:** Horizontal scaling across regions
- **Strong consistency:** ACID transactions globally
- **SQL interface:** Standard SQL queries
- **High availability:** 99.999% SLA
- **Automatic sharding:** Transparent data distribution

**Use Cases:**
- Global applications
- Financial services
- Gaming leaderboards
- Supply chain management

### Firestore
**What it is:** NoSQL document database for mobile and web applications

**Key Features:**
- **Real-time synchronization:** Live updates to connected clients
- **Offline support:** Local caching and sync when online
- **Multi-region replication:** Global data distribution
- **Security rules:** Declarative security at the document level
- **ACID transactions:** Strong consistency guarantees

**Use Cases:**
- Mobile applications
- Real-time chat applications
- Social media platforms
- Collaborative applications

### Cloud Bigtable
**What it is:** NoSQL wide-column database for large analytical workloads

**Key Features:**
- **High throughput:** Millions of operations per second
- **Low latency:** Single-digit millisecond latency
- **Scalability:** Petabyte-scale storage
- **Integration:** Works with Hadoop, Spark, and HBase APIs

**Use Cases:**
- Time-series data
- IoT applications
- Financial data analysis
- Real-time personalization

### BigQuery (Data Warehouse)
**What it is:** Serverless, highly scalable data warehouse

**Key Features:**
- **Serverless:** No infrastructure management
- **SQL interface:** Standard SQL queries
- **Real-time analytics:** Stream data for real-time insights
- **Machine learning:** Built-in ML capabilities (BigQuery ML)
- **Data sharing:** Secure data sharing across organizations

**Use Cases:**
- Business intelligence and reporting
- Data analytics and data science
- Log analysis and monitoring
- Real-time dashboards

## Networking Services

### Virtual Private Cloud (VPC)
**What it is:** Software-defined networking for GCP resources

**Key Features:**
- **Global scope:** Single VPC spans all regions
- **Subnets:** Regional IP address ranges
- **Firewall rules:** Control traffic to/from instances
- **Private Google Access:** Access Google services without external IPs
- **VPC peering:** Connect VPCs privately

**Components:**
- **Subnets:** Segment your network
- **Routes:** Define traffic paths
- **Firewall rules:** Security policies
- **VPN gateways:** Hybrid connectivity

### Cloud Load Balancing
**What it is:** Distribute incoming traffic across multiple backend instances

**Types:**
- **Global HTTP(S):** Global web applications
- **Global SSL Proxy:** Global SSL/TLS traffic
- **Global TCP Proxy:** Global TCP traffic
- **Regional Network:** Regional TCP/UDP traffic
- **Regional Internal:** Internal load balancing

**Key Features:**
- **Global reach:** Single IP serves globally
- **Auto-scaling:** Scale backends automatically
- **Health checks:** Route traffic only to healthy instances
- **SSL termination:** Handle SSL/TLS encryption

### Cloud CDN
**What it is:** Global content delivery network

**Key Features:**
- **Global edge locations:** Cache content close to users
- **Integration:** Works with Load Balancing
- **Cache invalidation:** Programmatic cache control
- **Security:** DDoS protection and SSL support

**Use Cases:**
- Website acceleration
- Video streaming
- Software distribution
- API acceleration

### Cloud Interconnect and VPN
**What it is:** Hybrid connectivity options

**Cloud VPN:**
- **Site-to-site VPN:** Connect on-premises to GCP
- **IPsec encryption:** Secure tunnel over internet
- **High availability:** 99.9% SLA with redundant gateways

**Cloud Interconnect:**
- **Dedicated Interconnect:** Direct physical connection
- **Partner Interconnect:** Connection through service provider
- **Higher bandwidth:** Up to 200 Gbps
- **Lower latency:** Direct connection to Google network

## AI and Machine Learning Services

### Vertex AI
**What it is:** Unified ML platform for building, deploying, and scaling ML models

**Key Features:**
- **AutoML:** Build custom models without coding
- **Custom training:** Train models with your own code
- **Model deployment:** Serve models with managed endpoints
- **MLOps:** End-to-end ML workflow management
- **Pre-built algorithms:** Ready-to-use ML components

**Use Cases:**
- Custom machine learning models
- Computer vision applications
- Natural language processing
- Recommendation systems

### Pre-trained AI APIs
**Vision AI:**
- **Image analysis:** Detect objects, faces, text in images
- **Video analysis:** Analyze video content
- **Use cases:** Content moderation, product search, accessibility

**Natural Language AI:**
- **Text analysis:** Sentiment, entity, syntax analysis
- **Translation:** 100+ language pairs
- **Use cases:** Customer feedback analysis, content localization

**Speech-to-Text and Text-to-Speech:**
- **Audio transcription:** Convert speech to text
- **Voice synthesis:** Convert text to natural speech
- **Use cases:** Call center analytics, voice assistants

**Document AI:**
- **Document processing:** Extract data from documents
- **Form parsing:** Process structured and unstructured forms
- **Use cases:** Invoice processing, document digitization

### Translation AI
**What it is:** Real-time language translation service

**Key Features:**
- **100+ languages:** Comprehensive language support
- **AutoML Translation:** Custom translation models
- **Batch translation:** Process large documents
- **Real-time translation:** Live translation for applications

## Operations and Monitoring

### Cloud Monitoring
**What it is:** Infrastructure and application monitoring service

**Key Features:**
- **Metrics collection:** System and custom metrics
- **Alerting:** Proactive notifications
- **Dashboards:** Visual monitoring displays
- **Uptime monitoring:** Website and API availability
- **Integration:** Works with third-party tools

### Cloud Logging
**What it is:** Centralized logging service for applications and infrastructure

**Key Features:**
- **Log ingestion:** Collect logs from multiple sources
- **Search and filter:** Query logs with advanced filters
- **Export:** Send logs to BigQuery, Cloud Storage, Pub/Sub
- **Retention:** Configurable log retention policies

### Cloud Trace
**What it is:** Distributed tracing system for application performance

**Key Features:**
- **Request tracing:** Track requests across services
- **Performance insights:** Identify bottlenecks
- **Integration:** Works with App Engine, GKE, Compute Engine
- **Sampling:** Configurable trace sampling rates

### Error Reporting
**What it is:** Real-time error monitoring and alerting

**Key Features:**
- **Error aggregation:** Group similar errors
- **Alerting:** Notifications for new errors
- **Integration:** Works with multiple languages and platforms
- **Stack traces:** Detailed error information