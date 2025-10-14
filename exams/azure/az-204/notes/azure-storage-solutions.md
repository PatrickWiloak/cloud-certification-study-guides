# Develop for Azure Storage

## Overview
This domain covers developing solutions using Azure Cosmos DB and Azure Blob Storage. It represents 15-20% of the exam and focuses on data storage operations, SDK usage, and storage management strategies.

**[📖 Azure Storage Documentation](https://learn.microsoft.com/en-us/azure/storage/)** - Complete Azure Storage documentation
**[📖 Azure Cosmos DB Documentation](https://learn.microsoft.com/en-us/azure/cosmos-db/)** - Official Cosmos DB documentation and guides

## Key Topics

### Cosmos DB Storage Solutions
- API and SDK selection for different scenarios
- Partitioning schemes and partition key implementation
- Data and container operations using SDKs
- Consistency level configuration for operations
- Change feed notification management

**[📖 Cosmos DB Partitioning](https://learn.microsoft.com/en-us/azure/cosmos-db/partitioning-overview)** - Partitioning and partition key strategies
**[📖 Cosmos DB Consistency Levels](https://learn.microsoft.com/en-us/azure/cosmos-db/consistency-levels)** - Understanding consistency trade-offs
**[📖 Cosmos DB Change Feed](https://learn.microsoft.com/en-us/azure/cosmos-db/change-feed)** - Change feed processing patterns

### Blob Storage Solutions
- Moving items between storage accounts and containers
- Setting and retrieving properties and metadata
- Data operations using appropriate SDKs
- Storage policies and data lifecycle management
- Data archiving and retention implementation

**[📖 Azure Blob Storage Documentation](https://learn.microsoft.com/en-us/azure/storage/blobs/)** - Complete Blob Storage guide
**[📖 Blob Storage Lifecycle Management](https://learn.microsoft.com/en-us/azure/storage/blobs/lifecycle-management-overview)** - Automated lifecycle policies
**[📖 Blob Storage Access Tiers](https://learn.microsoft.com/en-us/azure/storage/blobs/access-tiers-overview)** - Hot, Cool, and Archive tier optimization

## Services Reference

### Core Services
- Azure Cosmos DB (SQL, MongoDB, Cassandra, Gremlin, Table APIs)
- Azure Blob Storage
- Azure Storage SDK for .NET, Python, JavaScript, Java

**[📖 Cosmos DB .NET SDK](https://learn.microsoft.com/en-us/azure/cosmos-db/nosql/sdk-dotnet-v3)** - .NET SDK v3 for Cosmos DB
**[📖 Azure Storage Blobs Client Library](https://learn.microsoft.com/en-us/dotnet/api/overview/azure/storage.blobs-readme)** - Blob Storage SDK for .NET

### Supporting Services
- Azure Storage Explorer
- Azure Data Lake Storage Gen2
- Azure Content Delivery Network (CDN)

**[📖 Azure Data Lake Storage Gen2](https://learn.microsoft.com/en-us/azure/storage/blobs/data-lake-storage-introduction)** - Hierarchical namespace for big data analytics

## Best Practices

### Cosmos DB Development
- Choose appropriate consistency level based on requirements
- Design effective partition keys to avoid hot partitions
- Use point reads when possible for best performance
- Implement retry logic for transient failures
- Monitor RU consumption and optimize queries

### Blob Storage Management
- Use appropriate storage tier (Hot, Cool, Archive)
- Implement lifecycle management policies for cost optimization
- Use SAS tokens for secure, time-limited access
- Enable soft delete for data protection
- Leverage blob indexing for efficient searching

### Performance Optimization
- Use async operations for better throughput
- Batch operations when possible
- Implement proper connection pooling
- Cache frequently accessed data
- Choose appropriate consistency level for Cosmos DB

## Common Scenarios

### Data Storage Patterns
- Document storage with Cosmos DB SQL API
- Time-series data with appropriate partition keys
- Large file storage and retrieval with Blob Storage
- Hierarchical data storage with Data Lake Storage Gen2

### Integration Scenarios
- Change feed processing for real-time updates
- Blob trigger functions for automated processing
- Data replication across regions for disaster recovery
- CDN integration for static content delivery

## Study Tips

- Practice using Cosmos DB SDKs for CRUD operations
- Understand partition key selection and impact on performance
- Learn the five consistency levels and trade-offs
- Hands-on experience with blob lifecycle management policies
- Understand different blob types (Block, Append, Page)
- Practice SAS token generation and usage
- Study change feed implementation patterns
- Learn blob metadata and properties management
