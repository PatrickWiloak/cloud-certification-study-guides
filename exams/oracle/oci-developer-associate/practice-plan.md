# OCI Developer Associate - Practice Plan

A comprehensive 6-week study plan for the Oracle Cloud Infrastructure Developer Associate certification (1Z0-1084-24).

## Week 1-2: Cloud Native Development

### Week 1: Functions and API Gateway

**Day 1-2: Functions Basics**
- [ ] Create OCI account / Use existing
- [ ] Install Fn CLI
- [ ] Deploy first function (Hello World)
- [ ] Test function invocation
- [ ] **Lab**: Create function to process JSON
- [ ] **Practice**: 10 Functions quiz questions

**Day 3-4: Functions with OCI Services**
- [ ] Configure resource principal
- [ ] Access Object Storage from function
- [ ] Trigger function from Object Storage event
- [ ] **Lab**: Image thumbnail generation function
- [ ] **Practice**: 15 Functions integration questions

**Day 5-7: API Gateway**
- [ ] Create API Gateway
- [ ] Deploy API with multiple routes
- [ ] Configure authentication (JWT)
- [ ] Set up rate limiting and CORS
- [ ] **Lab**: Build REST API with Functions backend
- [ ] **Practice**: 20 API Gateway questions

### Week 2: Containers (OKE and OCIR)

**Day 1-3: Container Engine (OKE)**
- [ ] Create OKE cluster
- [ ] Deploy application to OKE
- [ ] Configure kubectl
- [ ] Create services and ingress
- [ ] **Lab**: Deploy multi-tier app on OKE
- [ ] **Practice**: 20 OKE questions

**Day 4-5: Container Registry (OCIR)**
- [ ] Push image to OCIR
- [ ] Pull image from OCIR
- [ ] Configure IAM policies
- [ ] **Lab**: CI/CD with OCIR
- [ ] **Practice**: 15 OCIR questions

**Day 6-7: Review and Practice**
- [ ] Review all Week 1-2 notes
- [ ] Complete hands-on labs again
- [ ] Take practice test (40 questions)
- [ ] **Practice**: Focus on weak areas

## Week 3-4: Application Deployment and Integration

### Week 3: Messaging and Deployment

**Day 1-2: Queue Service**
- [ ] Create queue
- [ ] Send and receive messages
- [ ] Configure visibility timeout
- [ ] **Lab**: Async task processing with Queue
- [ ] **Practice**: 10 Queue questions

**Day 3-4: Streaming**
- [ ] Create stream
- [ ] Produce messages to stream
- [ ] Consume with cursor
- [ ] Configure consumer groups
- [ ] **Lab**: Event pipeline with Streaming
- [ ] **Practice**: 15 Streaming questions

**Day 5-7: Deployment Strategies**
- [ ] Blue-green deployment on OKE
- [ ] Canary deployment (Istio)
- [ ] Rolling updates
- [ ] **Lab**: Implement all three strategies
- [ ] **Practice**: 20 deployment questions

### Week 4: Service Mesh and Integration

**Day 1-3: Service Mesh (Istio)**
- [ ] Install Istio on OKE
- [ ] Configure traffic management
- [ ] Set up circuit breaker
- [ ] Enable distributed tracing
- [ ] **Lab**: Microservices with Istio
- [ ] **Practice**: 15 Service Mesh questions

**Day 4-5: Events and Integration**
- [ ] Create event rules
- [ ] Configure Service Connector Hub
- [ ] **Lab**: Event-driven architecture
- [ ] **Practice**: 15 integration questions

**Day 6-7: Review and Practice**
- [ ] Review Week 3-4 notes
- [ ] Practice test (40 questions)
- [ ] Hands-on review
- [ ] **Practice**: Focus on weak areas

## Week 5: DevOps and Monitoring

### Days 1-2: DevOps Service
- [ ] Create DevOps project
- [ ] Set up code repository
- [ ] Create build pipeline
- [ ] Create deployment pipeline
- [ ] Configure triggers
- [ ] **Lab**: Complete CI/CD pipeline
- [ ] **Practice**: 25 DevOps questions

### Days 3-4: Resource Manager
- [ ] Write Terraform configuration
- [ ] Create Resource Manager stack
- [ ] Plan and apply changes
- [ ] Detect drift
- [ ] **Lab**: Deploy infrastructure with IaC
- [ ] **Practice**: 15 Resource Manager questions

### Days 5-7: Monitoring and Logging
- [ ] Configure APM domain
- [ ] Instrument application
- [ ] View distributed traces
- [ ] Create custom logs
- [ ] Write Log Analytics queries
- [ ] **Lab**: Complete observability setup
- [ ] **Practice**: 20 monitoring questions

## Week 6: Final Preparation

### Days 1-2: Comprehensive Review
- [ ] Review all study notes
- [ ] Re-do all hands-on labs
- [ ] Focus on weak topics
- [ ] **Practice**: 50 mixed questions

### Days 3-4: Practice Exams
- [ ] Practice exam #1 (60 questions, timed)
- [ ] Review incorrect answers thoroughly
- [ ] Practice exam #2 (60 questions, timed)
- [ ] **Target**: 80%+ score

### Days 5-6: Final Polish
- [ ] Quick review of all topics
- [ ] Focus on scenarios and use cases
- [ ] Review exam objectives
- [ ] **Practice**: 30 scenario-based questions

### Day 7: Pre-Exam
- [ ] Light review only
- [ ] No new material
- [ ] Rest and relaxation
- [ ] Prepare exam environment

## Hands-on Labs Checklist

### Functions
- [ ] Create and deploy function
- [ ] Trigger function from Object Storage
- [ ] Access OCI services from function
- [ ] Image processing function

### API Gateway
- [ ] Deploy API with multiple routes
- [ ] Configure JWT authentication
- [ ] Set up rate limiting
- [ ] Integrate with Functions backend

### OKE
- [ ] Create OKE cluster
- [ ] Deploy containerized application
- [ ] Configure services and ingress
- [ ] Implement autoscaling

### OCIR
- [ ] Push images to registry
- [ ] Configure access policies
- [ ] Pull images in OKE

### Messaging
- [ ] Send/receive messages (Queue)
- [ ] Produce/consume (Streaming)
- [ ] Consumer groups

### Deployment
- [ ] Blue-green deployment
- [ ] Canary deployment
- [ ] Rolling update

### DevOps
- [ ] Complete CI/CD pipeline
- [ ] Build and deploy automatically
- [ ] Configure approval gates

### Monitoring
- [ ] Instrument application with APM
- [ ] View distributed traces
- [ ] Create Log Analytics queries
- [ ] Build custom dashboard

## Key Topics to Master

### Critical (Must Know)
- Functions: Resource principal, event triggers
- API Gateway: Authentication, rate limiting
- OKE: Deployments, services, kubectl
- OCIR: Image management, IAM
- Queue vs Streaming: Use cases, differences
- DevOps: Build pipeline, deployment pipeline
- APM: Distributed tracing, instrumentation

### Important (Should Know)
- Deployment strategies (blue-green, canary)
- Service Mesh (Istio): Traffic management
- Events and Service Connector
- Resource Manager: Terraform, stacks
- Logging: Service logs, custom logs

### Good to Know
- Advanced Istio features
- Complex event patterns
- Advanced APM queries
- Drift detection

## Exam Strategy

**Time Management**
- 90 minutes for 60 questions = 1.5 min/question
- First pass: Answer known questions (60 min)
- Second pass: Harder questions (20 min)
- Final review: Check all answers (10 min)

**Question Approach**
- Read carefully (keywords matter)
- Eliminate wrong answers
- Choose "most correct" answer
- Flag uncertain questions

**Common Question Patterns**
- "Which service for..."  (Choose right service)
- "How to implement..."  (Implementation steps)
- "What happens when..." (Understand behavior)
- "Best practice for..." (Architecture decisions)

## Resources

### Official
- [ ] Oracle Learning Paths (mylearn.oracle.com)
- [ ] OCI Documentation
- [ ] Hands-on Labs (Oracle LiveLabs)
- [ ] Sample questions (Oracle)

### Practice
- [ ] Study notes (this repo)
- [ ] Hands-on labs (above checklist)
- [ ] Practice exams (minimum 200 questions)
- [ ] Scenario-based questions

### Tools
- [ ] OCI CLI installed
- [ ] kubectl configured
- [ ] Fn CLI installed
- [ ] Docker installed
- [ ] Terraform installed

## Week Before Exam

- [ ] Take final practice exam (timed, 60 questions)
- [ ] Score 85%+ consistently
- [ ] Review flagged topics
- [ ] No new labs (review only)
- [ ] Rest well (8 hours sleep)

## Exam Day

- [ ] Good night's sleep
- [ ] Healthy breakfast
- [ ] Arrive/login 15 minutes early
- [ ] Read questions carefully
- [ ] Manage time effectively
- [ ] Stay calm and confident

---

**Good luck with your OCI Developer Associate certification!**

Focus on hands-on practice - the exam tests practical knowledge!
