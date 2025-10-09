# Microsoft Azure Developer Associate (AZ-204)

## Exam Overview

The Azure Developer Associate certification validates skills in designing, building, testing, and maintaining cloud applications and services on Microsoft Azure. Azure Developers participate in all phases of cloud development from requirements definition and design, to development, deployment, and maintenance.

**Exam Details:**
- **Exam Code:** AZ-204
- **Duration:** 180 minutes
- **Number of Questions:** 40-60 questions
- **Passing Score:** 700 out of 1000
- **Question Types:** Multiple choice, multiple select, drag and drop, hot area, case studies, labs
- **Cost:** $165 USD
- **Prerequisites:** 1-2 years development experience and experience with Azure

## Exam Domains

### 1. Develop Azure Compute Solutions (25-30%)
- **Implement IaaS Solutions**
  - Provision VMs
  - Configure, validate, and deploy ARM templates
  - Configure container images for solutions
  - Publish an image to Azure Container Registry
  - Run containers by using Azure Container Instance

- **Create Azure App Service Web Apps**
  - Create an Azure App Service Web App
  - Enable diagnostics logging
  - Deploy code to a web app
  - Configure web app settings including SSL, API settings, and connection strings
  - Implement autoscaling rules including scheduled autoscaling and scaling by operational or system metrics

- **Implement Azure Functions**
  - Create and deploy Azure Functions apps
  - Implement input and output bindings for a function
  - Implement function triggers by using data operations, timers, and webhooks
  - Implement Azure Durable Functions
  - Implement custom handlers

### 2. Develop for Azure Storage (15-20%)
- **Develop Solutions that Use Cosmos DB Storage**
  - Select the appropriate API and SDK for a solution
  - Implement partitioning schemes and partition keys
  - Perform operations on data and Cosmos DB containers
  - Set the appropriate consistency level for operations
  - Manage change feed notifications

- **Develop Solutions that Use Blob Storage**
  - Move items in Blob storage between storage accounts or containers
  - Set and retrieve properties and metadata
  - Perform operations on data by using the appropriate SDK
  - Implement storage policies and data archiving and retention

### 3. Implement Azure Security (20-25%)
- **Implement User Authentication and Authorization**
  - Authenticate and authorize users by using the Microsoft Identity platform
  - Authenticate and authorize users and apps by using Azure Active Directory
  - Create and implement shared access signatures
  - Implement solutions that interact with Microsoft Graph

- **Implement Secure Cloud Solutions**
  - Secure app configuration data by using App Configuration or Azure Key Vault
  - Develop code that uses keys, secrets, and certificates stored in Azure Key Vault
  - Implement Managed Identities for Azure resources

### 4. Monitor, Troubleshoot, and Optimize Azure Solutions (15-20%)
- **Integrate Caching and Content Delivery within Solutions**
  - Configure cache and expiration policies for Azure Redis Cache
  - Implement secure and optimized application cache patterns including data sizing, connections, encryption, and expiration
  - Implement Azure Content Delivery Network endpoints and profiles

- **Instrument Solutions to Support Monitoring and Logging**
  - Configure an app or service to use Application Insights
  - Analyze and troubleshoot solutions by using Azure Monitor
  - Implement Application Insights web tests and alerts

### 5. Connect to and Consume Azure Services and Third-party Services (15-20%)
- **Implement API Management**
  - Create an APIM instance
  - Configure authentication for APIs
  - Define policies for APIs

- **Develop Event-based Solutions**
  - Implement solutions that use Azure Event Grid
  - Implement solutions that use Azure Event Hubs
  - Implement solutions that use Azure Service Bus

- **Develop Message-based Solutions**
  - Implement solutions that use Azure Service Bus
  - Implement solutions that use Azure Queue Storage queues

## Skills Measured in Detail

### Development and Deployment
- **Application Architecture:** Microservices, serverless, containerized applications
- **CI/CD Pipelines:** Azure DevOps, GitHub Actions, deployment strategies
- **Infrastructure as Code:** ARM templates, Bicep, Terraform integration
- **Container Development:** Docker, Azure Container Registry, AKS deployment

### Data and Storage Solutions
- **Cosmos DB Development:** API selection, partitioning strategies, consistency models
- **Blob Storage Operations:** SDK usage, lifecycle management, performance optimization
- **Caching Strategies:** Redis implementation, cache patterns, performance tuning

### Security Implementation
- **Identity Integration:** Azure AD, OAuth 2.0, OpenID Connect
- **Secret Management:** Key Vault integration, managed identities
- **API Security:** Authentication, authorization, token validation

### Monitoring and Optimization
- **Application Performance:** Application Insights, custom telemetry
- **Diagnostics:** Logging strategies, error tracking, performance monitoring
- **Optimization:** Cost optimization, performance tuning, scalability planning

### Integration Patterns
- **Event-Driven Architecture:** Event Grid, Event Hubs, Service Bus
- **API Management:** Gateway patterns, policy implementation, versioning
- **Message Queuing:** Reliable messaging, dead letter handling, scaling

## Programming Languages and SDKs

### Supported Languages
- **C# (.NET Core/.NET 5+)**
- **Python**
- **JavaScript/TypeScript (Node.js)**
- **Java**

### Key SDKs and Libraries
- **Azure SDK for .NET**
- **Azure SDK for Python**
- **Azure SDK for JavaScript**
- **Azure SDK for Java**
- **Azure Functions runtime libraries**
- **Azure Storage SDKs**
- **Azure Cosmos DB SDKs**

## Study Tips

### Recommended Study Timeline: 10-14 weeks
1. **Weeks 1-2:** Azure development fundamentals and tooling
2. **Weeks 3-4:** App Service and Azure Functions development
3. **Weeks 5-6:** Storage solutions (Cosmos DB, Blob Storage)
4. **Weeks 7-8:** Security and identity implementation
5. **Weeks 9-10:** Monitoring, caching, and optimization
6. **Weeks 11-12:** Event-driven and messaging solutions
7. **Weeks 13-14:** Practice exams and comprehensive review

### Key Study Resources
- **Microsoft Learn Learning Paths:**
  - AZ-204: Create Azure App Service web apps
  - AZ-204: Implement Azure Functions
  - AZ-204: Develop solutions that use blob storage
  - AZ-204: Develop solutions that use Cosmos DB storage
  - AZ-204: Implement infrastructure as a service solutions
  - AZ-204: Implement user authentication and authorization
  - AZ-204: Implement secure cloud solutions
  - AZ-204: Implement API Management
  - AZ-204: Develop event-based solutions
  - AZ-204: Develop message-based solutions

### Development Environment Setup
- **Local Development:** Visual Studio/VS Code, Azure CLI, Azure Functions Core Tools
- **Azure Subscription:** Pay-as-you-go or developer subscription
- **Source Control:** Git, GitHub/Azure DevOps
- **Containers:** Docker Desktop for local development

### Hands-on Practice Requirements
- **Development Experience:** 1-2 years with Azure and cloud development
- **SDK Familiarity:** Hands-on experience with Azure SDKs
- **Real Applications:** Build and deploy complete applications
- **Integration Scenarios:** Event-driven and microservices architectures

### Exam Strategy
- **Code-focused:** Expect questions about specific SDK usage and code patterns
- **Scenario-based:** Understand when to use different services and patterns
- **Performance Labs:** Practice with Azure portal and development tools
- **Troubleshooting:** Know how to diagnose and resolve common issues

### Common Gotchas
- **Cosmos DB Consistency:** Understand different consistency levels and trade-offs
- **Function Bindings:** Know input/output binding configurations
- **Storage Access:** SAS tokens, managed identities, and access patterns
- **Event Processing:** Differences between Event Grid, Event Hubs, and Service Bus
- **Authentication Flows:** OAuth 2.0, OpenID Connect, and Azure AD integration

## Hands-on Lab Areas

### Development Labs
- Create and deploy Azure Functions with various triggers
- Build and deploy containerized applications
- Implement CI/CD pipelines with Azure DevOps
- Develop multi-tier applications with App Service

### Storage Labs
- Implement Cosmos DB solutions with different APIs
- Build blob storage applications with lifecycle management
- Implement caching with Azure Redis Cache
- Create event-driven storage solutions

### Security Labs
- Integrate Azure AD authentication in web applications
- Implement Key Vault for secret management
- Configure managed identities for applications
- Implement API authentication and authorization

### Integration Labs
- Build event-driven solutions with Event Grid
- Implement message queuing with Service Bus
- Create API Management solutions
- Develop hybrid integration scenarios

### Monitoring Labs
- Configure Application Insights for applications
- Implement custom telemetry and logging
- Set up alerts and dashboards
- Performance testing and optimization

## Code Examples and Patterns

### Azure Functions Development
```csharp
[FunctionName("HttpTriggerFunction")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req,
    [CosmosDB(databaseName: "mydb", collectionName: "items", ConnectionStringSetting = "CosmosDB")] IAsyncCollector<dynamic> documentsOut,
    ILogger log)
{
    // Function implementation
}
```

### Cosmos DB Operations
```csharp
// Create item
ItemResponse<Product> response = await container.CreateItemAsync(product, new PartitionKey(product.CategoryId));

// Query items
FeedIterator<Product> query = container.GetItemQueryIterator<Product>("SELECT * FROM c WHERE c.category = 'electronics'");
```

### Event Grid Publishing
```csharp
var events = new List<EventGridEvent>
{
    new EventGridEvent("subject", "event-type", "1.0", new { Property = "value" })
};
await client.SendEventsAsync(events);
```

## 📚 Comprehensive Study Resources

**👉 [Complete Azure Study Resources Guide](../../../.templates/resources-azure.md)**

For detailed information on courses, practice tests, hands-on labs, communities, and more, see our comprehensive Azure study resources guide which includes:
- Official Microsoft Learn paths (FREE)
- Top-rated video courses with specific instructors
- Practice test platforms with pricing and comparisons
- Hands-on lab environments and free tier details
- Community forums and study groups
- Essential tools and Azure CLI resources
- Pro tips and budget-friendly study strategies

### Quick Links (AZ-204 Specific)
- **[AZ-204 Official Exam Page](https://learn.microsoft.com/en-us/certifications/exams/az-204/)** - Registration and exam details
- **[Microsoft Learn - AZ-204 Learning Path](https://learn.microsoft.com/en-us/certifications/azure-developer/)** - FREE official study path
- **[Azure Developer Documentation](https://docs.microsoft.com/en-us/azure/developer/)** - Developer guides
- **[Azure SDK Documentation](https://docs.microsoft.com/en-us/azure/developer/sdk/)** - SDK references
- **[Azure Code Samples](https://docs.microsoft.com/en-us/samples/browse/?products=azure)** - Example code
- **[Azure Functions Documentation](https://docs.microsoft.com/en-us/azure/azure-functions/)** - Functions reference
- **[Azure Free Account](https://azure.microsoft.com/en-us/free/)** - $200 free credit for hands-on practice

## Prerequisites and Next Steps

### Prerequisites
- Experience with Azure fundamentals (AZ-900 helpful)
- 1-2 years of development experience
- Proficiency in at least one Azure-supported programming language
- Understanding of cloud development patterns

### Career Path
- **Next Certifications:** AZ-305 (Solutions Architect Expert), specialized role-based certs
- **Role Focus:** Cloud Developer, Application Developer, DevOps Engineer
- **Skills Development:** DevOps practices, microservices, cloud-native development

Remember: AZ-204 is a hands-on developer exam requiring practical coding experience with Azure services. Focus on building real applications and understanding service integration patterns.