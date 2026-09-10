# Scalability

## 1. Purpose

The Job Discovery Platform is expected to grow from a small V1 product into a larger job discovery system.

The initial system should remain simple.

The goal is not to build a highly distributed system on Day 1.

The goal is to create an architecture that can grow when real product usage creates real bottlenecks.

The main scalability challenge is not only user traffic.

The platform also has to process:

- Search requests
- Search queries
- External URLs
- Public web pages
- Raw documents
- Job extraction
- Validation
- Deduplication
- Freshness checks
- Search and ranking

Therefore, discovery workload and user-facing traffic should be treated as different workloads.

---

# 2. Scalability Principles

The system should follow these principles:

- Build simple first.
- Measure before scaling.
- Scale the bottleneck instead of the whole system.
- Keep the API layer as stateless as possible.
- Separate user requests from long-running discovery work.
- Make processing retryable.
- Make processing idempotent.
- Control external request rates.
- Isolate failures.
- Avoid unnecessary infrastructure.
- Keep source adapters independent.
- Keep the domain model stable.

---

# 3. V1 Architecture

The initial architecture is:

    Frontend
       ↓
    Backend API
       ↓
    Spring Boot Modular Monolith
       ↓
    Async Processing
       ↓
    PostgreSQL

The frontend is a separate application.

The backend is a single deployable application containing the discovery pipeline.

The backend is the primary scalability concern during V1.

---

# 4. Two Different Workloads

The platform has two major workloads.

## User Workload

Examples:

- Search
- View opportunities
- Open details
- Apply filters
- Sort results

This workload should remain fast.

## Discovery Workload

Examples:

- Generate queries
- Search external providers
- Process URLs
- Fetch pages
- Parse HTML
- Extract jobs
- Validate
- Deduplicate
- Store results

This workload can be slow and unpredictable.

The architecture should keep these workloads separate.

    User Request
         ↓
    Fast API
         ↓
    Discovery Task
         ↓
    Background Processing

---

# 5. Frontend Scalability

The frontend should remain lightweight.

It should not perform expensive discovery operations.

The frontend should:

- Send search requests
- Poll or retrieve task status
- Request results
- Render opportunities
- Navigate to details

The frontend should not:

- Crawl websites
- Call search providers directly
- Parse external job pages
- Perform deduplication
- Run heavy processing

This keeps the frontend easy to scale independently later.

---

# 6. Stateless Backend API

The backend API should be as stateless as possible.

Important persistent state should be stored in:

    PostgreSQL

Future supporting systems may include:

    Redis
    Object Storage
    Search Index

The API should not depend on local server memory for important application state.

This allows multiple backend instances to be added later.

---

# 7. Horizontal API Scaling

When user traffic increases, multiple backend instances can be deployed.

Example:

    Frontend
       ↓
    Load Balancer
       ↓
    ┌───────────────┐
    │ Backend API 1 │
    │ Backend API 2 │
    │ Backend API 3 │
    └───────┬───────┘
            ↓
        PostgreSQL

Because the API is stateless, requests can be distributed between instances.

---

# 8. Discovery Workload Scaling

Discovery is more expensive than a normal API request.

One search can produce:

    1 User Search
        ↓
    Multiple Queries
        ↓
    Many URLs
        ↓
    Many Page Requests
        ↓
    Many Extraction Operations

Therefore discovery should not run completely inside the API request thread.

Preferred flow:

    API
      ↓
    Create Discovery Task
      ↓
    Background Worker
      ↓
    Discovery Pipeline
      ↓
    Database

---

# 9. V1 Async Processing

V1 can use:

    Spring Async
        +
    Controlled Thread Pool

This provides background processing without introducing a distributed queue.

The thread pool should have controlled limits.

The application should not create an unlimited number of threads.

---

# 10. When a Queue Becomes Necessary

A queue becomes useful when the discovery workload becomes large enough that a local thread pool is no longer sufficient.

Possible signals:

- Large number of simultaneous discovery tasks
- Long task queues
- Need for multiple worker instances
- More advanced retry requirements
- Need for durable task buffering
- Need for independent worker scaling

Possible future technologies:

- Kafka
- RabbitMQ
- Cloud-managed queues

The technology should be selected based on actual requirements.

Kafka is not required for V1.

---

# 11. Worker Scaling

When discovery workload increases:

    Discovery Queue
         ↓
    ┌───────────────┐
    │ Worker 1      │
    │ Worker 2      │
    │ Worker 3      │
    │ Worker N      │
    └───────┬───────┘
            ↓
    Discovery Pipeline

Workers can scale independently from API instances.

This is important because adding more API servers should not automatically create excessive external crawling.

---

# 12. Backpressure

The system must control how much work is sent to external sources.

Example:

    Many User Searches
           ↓
    Discovery Queue
           ↓
    Controlled Workers
           ↓
    External Sources

This prevents:

- Excessive provider requests
- Excessive website requests
- Database overload
- CPU overload
- Network overload

Backpressure becomes especially important when discovery volume increases.

---

# 13. External Source Rate Limits

Different sources may have different limits.

The system should support source-specific controls.

Conceptually:

    Source A
        → Controlled Request Rate

    Source B
        → Different Request Rate

    Search Provider
        → Provider-specific Limits

The exact limits must come from the source/provider requirements.

The system should not assume every source supports the same request rate.

---

# 14. Fetching Scalability

Page fetching is network-heavy.

At higher volume, fetching can become one of the largest workloads.

Future architecture:

    Fetch Queue
        ↓
    Fetch Workers
        ↓
    Public Web

Workers should support:

- Timeout
- Retry
- Backoff
- Rate limiting
- Failure isolation

A failed page should not block unrelated pages.

---

# 15. Extraction Scalability

Extraction has different resource requirements.

Simple extraction:

    HTML
      ↓
    Parser
      ↓
    Text

is relatively lightweight.

AI-assisted extraction can be more expensive.

Therefore AI processing should be selective.

Preferred approach:

    Page
      ↓
    Deterministic Extraction
      ↓
    Confidence Check
      ↓
    AI Only If Needed

This controls both compute and AI cost.

---

# 16. Database Scalability

PostgreSQL should remain the initial source of truth.

Before introducing another database, optimize PostgreSQL using:

- Proper indexes
- Query optimization
- Connection pooling
- Pagination
- Batch operations
- Appropriate schema design
- Data retention policies

The system should measure database performance before deciding to introduce additional infrastructure.

---

# 17. Important Database Indexes

Potential query dimensions include:

- City
- Company
- Opportunity type
- Event date
- Status
- Discovered time
- Last checked time

Composite indexes should be added based on actual query patterns.

Do not create indexes for every field without evidence.

Too many indexes can increase write cost and storage usage.

---

# 18. Database Connection Management

As backend instances increase, database connections can become a bottleneck.

For example:

    Backend 1
       ↓
    Connection Pool

    Backend 2
       ↓
    Connection Pool

    Backend 3
       ↓
    Connection Pool

The total number of database connections must remain within PostgreSQL capacity.

Adding more application instances does not mean unlimited database connections.

Connection pool sizes should be controlled.

---

# 19. Search Scalability

V1 search can use PostgreSQL.

This is sufficient for the initial product requirements.

As the opportunity dataset grows, search may require:

- Full-text search
- Advanced ranking
- Fuzzy matching
- Semantic search
- Large-scale filtering
- Low-latency search

Possible future technologies:

- PostgreSQL full-text search
- OpenSearch
- Elasticsearch

The decision should be based on:

- Dataset size
- Query complexity
- Search latency
- Ranking requirements

---

# 20. Caching

Redis can be introduced when repeated reads justify it.

Potential cache candidates:

- Popular searches
- Frequently viewed opportunities
- Normalized city values
- Source metadata
- Short-lived search results

The cache should not become the primary source of truth.

PostgreSQL remains authoritative.

---

# 21. Raw Document Storage

Raw HTML can become a significant storage workload.

Initially, raw documents can use the simplest appropriate storage approach.

At larger scale:

    Backend
       ↓
    Object Storage
       ↓
    Raw Documents

The database can retain:

- Document metadata
- Content hash
- Source URL
- Storage reference

Possible future storage:

- S3-compatible object storage
- Cloud object storage

---

# 22. Deduplication Scalability

Deduplication can become expensive as the number of opportunities grows.

Avoid:

    New Opportunity
          ↓
    Compare with Every Existing Opportunity

Instead, first identify likely candidates using:

- Company
- City
- Opportunity type
- Event date
- Normalized title

Then perform deeper comparison only on those candidates.

This reduces unnecessary comparisons.

---

# 23. Idempotency

Distributed systems retry.

A task or page may therefore be processed more than once.

The pipeline must safely handle this.

Possible mechanisms:

- Unique constraints
- Content hashes
- Opportunity identity keys
- Task IDs
- Normalized URLs

Example:

    Same Opportunity
         ↓
    Same Identity
         ↓
    Update Existing Record

instead of:

    Create Duplicate Record

---

# 24. Retry Strategy

Not every failure should be retried.

Examples:

    Temporary network error
        → Retry

    Timeout
        → Retry

    Rate limit
        → Backoff

    404
        → Usually do not repeatedly retry

    Authentication required
        → Do not bypass

    CAPTCHA
        → Do not bypass

Retries should have:

- Maximum attempts
- Backoff
- Failure tracking

---

# 25. Source Reliability

Different sources will have different reliability.

The platform should measure:

- Fetch success rate
- Extraction success rate
- Duplicate rate
- Expired result rate
- Completeness
- Freshness

This information can later contribute to source quality scoring.

---

# 26. Discovery Coverage

Adding more sources does not automatically improve the product.

The important metric is incremental useful coverage.

For each new source, measure:

    New Relevant Opportunities
            ↓
    Existing Relevant Opportunities

A source that adds little useful coverage may not justify its processing cost.

---

# 27. Scheduled Refresh

Future versions can periodically refresh stored opportunities.

Example:

    Active Opportunity
          ↓
    Scheduled Recheck
          ↓
    Fetch Source
          ↓
    Extract
          ↓
    Compare
          ↓
    Update

This can detect:

- Date changes
- Venue changes
- Job removal
- Event cancellation
- Description changes

---

# 28. Freshness-Based Processing Priority

Not every opportunity requires the same refresh frequency.

Possible future strategy:

    Event Soon
        → Higher Priority

    Event Far Away
        → Lower Priority

    Aging Opportunity
        → Lower Priority

    Expired Opportunity
        → No Regular Refresh

This reduces unnecessary processing.

---

# 29. Priority-Based Discovery

As workload increases, discovery tasks can be prioritized.

Example:

    HIGH
    User-triggered search

    MEDIUM
    Frequently requested search

    LOW
    Background refresh

This ensures interactive searches are not blocked by background processing.

---

# 30. Cost Scalability

Important future cost drivers include:

- Search provider usage
- External requests
- Compute
- AI extraction
- Database storage
- Object storage
- Search infrastructure

A useful future metric is:

    Total Processing Cost
    ----------------------
    Valid Opportunities Discovered

This helps determine whether a source or processing technique provides enough value.

---

# 31. AI Cost Control

If AI is introduced, the system should avoid sending every page to an AI model.

Preferred:

    All Pages
        ↓
    Cheap Deterministic Processing
        ↓
    Confidence Evaluation
        ↓
    AI Only For Uncertain Cases
        ↓
    Validation

This reduces:

- AI cost
- Processing time
- Dependency on external AI services

---

# 32. Frontend Scaling

Once frontend usage grows, it can scale independently.

Example:

    Users
      ↓
    CDN / Frontend Hosting
      ↓
    Frontend Application
      ↓
    Backend API

The frontend should remain independent from discovery worker scaling.

A spike in discovery processing should not require scaling the frontend.

---

# 33. API and Worker Separation

At higher scale:

    Users
      ↓
    Load Balancer
      ↓
    API Instances
      ↓
    Queue
      ↓
    Worker Instances
      ↓
    Discovery Pipeline

This creates an important separation:

    API Scaling
        ≠
    Worker Scaling

API instances can scale based on user traffic.

Workers can scale based on discovery workload.

---

# 34. Future Scaled Architecture

A possible future architecture is:

    ┌──────────────────────┐
    │        Users         │
    └──────────┬───────────┘
               ↓
    ┌──────────────────────┐
    │ Frontend / CDN       │
    └──────────┬───────────┘
               ↓
    ┌──────────────────────┐
    │ Load Balancer        │
    └──────────┬───────────┘
               ↓
    ┌──────────────────────┐
    │ API Instances        │
    └──────────┬───────────┘
               ↓
    ┌──────────────────────┐
    │ Discovery Queue      │
    └──────────┬───────────┘
               ↓
    ┌────────────────────────────┐
    │ Processing Workers         │
    │                            │
    │ Discovery                  │
    │ Fetching                   │
    │ Extraction                 │
    │ Validation                 │
    │ Deduplication              │
    └────────────┬───────────────┘
                 ↓
    ┌────────────────────────────┐
    │ Data Layer                 │
    │                            │
    │ PostgreSQL                 │
    │ Redis                      │
    │ Search Index               │
    │ Object Storage             │
    └────────────────────────────┘

This is a future architecture.

It is not required for V1.

---

# 35. When to Introduce More Infrastructure

Infrastructure should be introduced when a measurable problem appears.

Example:

    Problem:
    API latency

    Solution:
    Horizontal API scaling

    Problem:
    Discovery tasks take too long

    Solution:
    More workers / queue

    Problem:
    Repeated reads are expensive

    Solution:
    Redis

    Problem:
    Search becomes slow

    Solution:
    Search index or PostgreSQL optimization

    Problem:
    Raw documents consume too much database storage

    Solution:
    Object storage

    Problem:
    Worker coordination becomes difficult

    Solution:
    Distributed queue

The solution should follow the bottleneck.

---

# 36. What Not to Do

Do not introduce:

- Microservices
- Kafka
- Kubernetes
- Multiple databases
- Distributed infrastructure

only because the product might become large someday.

Premature infrastructure creates:

- More development work
- More operational cost
- More failure modes
- More debugging complexity

The product should earn its infrastructure.

---

# 37. Scalability Evolution

The expected evolution is:

## Stage 1 — V1

    Frontend
    Spring Boot
    PostgreSQL
    Async Thread Pool

## Stage 2 — Growing Product

    Frontend
    Multiple API Instances
    PostgreSQL
    Redis if needed
    Better indexing

## Stage 3 — Growing Discovery Workload

    API Instances
    Queue
    Multiple Workers
    PostgreSQL

## Stage 4 — Large Data Volume

    API
    Workers
    PostgreSQL
    Redis
    Search Index
    Object Storage

## Stage 5 — Large Distributed Platform

    Multiple Services
    Distributed Workers
    Advanced Messaging
    Multi-region Infrastructure

Each stage should be triggered by actual requirements.

---

# 38. Scalability Metrics

Track:

- API response time
- Discovery task duration
- Queue depth
- Worker utilization
- URLs processed per task
- Fetch success rate
- Fetch latency
- Extraction time
- Database query latency
- Database CPU
- Database storage
- Search latency
- Cache hit rate
- Cost per valid opportunity

These metrics should guide scaling decisions.

---

# 39. Core Scalability Principle

The platform should scale the processing pipeline independently from user-facing traffic.

The important separation is:

    User Traffic
        ≠
    Discovery Workload

A large number of web pages being processed should not make normal user API requests slow.

Similarly, a spike in user traffic should not automatically cause uncontrolled crawling of external websites.

---

# 40. Final Principle

The platform should evolve like this:

    Simple
       ↓
    Measured
       ↓
    Optimized
       ↓
    Decoupled
       ↓
    Distributed
       ↓
    Scaled

The goal is not maximum infrastructure.

The goal is a reliable job discovery system that can scale when real usage requires it.

The architecture should earn its complexity.
