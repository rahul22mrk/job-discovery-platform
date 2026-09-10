# Scalability

## 1. Purpose

The Job Discovery Platform may eventually process a large number of searches, URLs, source pages, and job opportunities.

However, scalability should be introduced based on actual system requirements.

The first goal is to build a correct and useful product.

The second goal is to scale the parts that become bottlenecks.

---

# 2. Scalability Principles

The system should follow these principles:

- Scale the bottleneck, not the entire system.
- Keep the architecture simple initially.
- Prefer stateless application instances.
- Separate network-heavy work from user-facing APIs.
- Make processing retryable.
- Make processing idempotent.
- Avoid unnecessary synchronous processing.
- Cache repeated work where useful.
- Measure before introducing infrastructure.
- Keep source adapters independent.

---

# 3. What Can Become a Bottleneck?

The main workload areas are different from a traditional CRUD application.

Potential bottlenecks include:

1. Search provider requests
2. External page fetching
3. HTML processing
4. Job extraction
5. AI-assisted extraction
6. Deduplication
7. Database writes
8. Search queries
9. Storage of raw documents
10. User search traffic

The architecture should allow these areas to scale independently when necessary.

---

# 4. V1 Scale

V1 does not need distributed infrastructure.

Initial architecture:

    Backend
       ↓
    Async Thread Pool
       ↓
    PostgreSQL

This is intentionally simple.

The goal is to validate:

- Discovery quality
- Extraction quality
- Relevance
- Freshness
- Deduplication

before optimizing infrastructure.

---

# 5. Stateless API

The backend API should remain as stateless as possible.

A request should not depend on local server memory for important persistent state.

Persistent state belongs in:

    PostgreSQL
    +
    Future Cache / Storage

This allows multiple backend instances to be added later.

---

# 6. Horizontal Scaling

When API traffic increases:

    Load Balancer
         ↓
    ┌──────────┐
    │ Backend 1│
    ├──────────┤
    │ Backend 2│
    ├──────────┤
    │ Backend 3│
    └──────────┘
         ↓
    PostgreSQL

The API layer can then scale horizontally.

This works best when backend instances remain stateless.

---

# 7. Discovery Workload Scaling

Discovery is different from normal API traffic.

A single search may trigger:

    Multiple queries
        ↓
    Many URLs
        ↓
    Many HTTP requests
        ↓
    Many extraction operations

Therefore discovery should not run directly inside the API request thread.

Instead:

    API
      ↓
    Discovery Task
      ↓
    Worker
      ↓
    Discovery Pipeline

---

# 8. Worker Scaling

As discovery volume grows:

    Discovery Queue
         ↓
    ┌─────────────┐
    │ Worker 1    │
    ├─────────────┤
    │ Worker 2    │
    ├─────────────┤
    │ Worker 3    │
    └─────────────┘

Workers can process discovery tasks independently.

The number of workers can increase according to workload.

---

# 9. Queue Introduction

A queue becomes useful when:

- Discovery tasks increase
- Processing becomes long-running
- Multiple workers are needed
- Retry handling becomes important
- Backpressure is required

Possible future technologies:

- Kafka
- RabbitMQ
- Cloud-managed queues

The technology should be selected based on actual requirements.

Kafka is not required for V1.

---

# 10. Backpressure

External sources have limits.

The system should not send unlimited requests just because internal traffic increases.

Example:

    User Traffic
         ↓
    Discovery Queue
         ↓
    Controlled Workers
         ↓
    External Sources

The queue and worker limits can protect:

- Search providers
- Source websites
- Database
- CPU
- Network

---

# 11. Rate Limiting

Different sources may have different request limits.

The platform should support source-specific limits.

Example:

    Search Provider
        10 requests/sec

    Source A
        1 request/sec

    Source B
        5 requests/sec

The exact limits should come from provider/source requirements.

The system should never assume that all sources can handle the same request rate.

---

# 12. Fetching Scalability

Page fetching is network-heavy.

As volume increases, fetching can become a major bottleneck.

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
- Rate limits
- Failure isolation

---

# 13. Extraction Scalability

Extraction can be CPU-intensive.

Basic HTML extraction may be relatively cheap.

AI-assisted extraction can be significantly more expensive.

Therefore AI processing should be isolated.

Possible flow:

    Normal Extraction
         ↓
    Successful
         ↓
    Store

If uncertain:

    AI Extraction
         ↓
    Validation
         ↓
    Store

AI should be used selectively rather than automatically for every page.

---

# 14. Database Scalability

PostgreSQL should be the initial source of truth.

Before introducing another database, optimize PostgreSQL using:

- Proper indexes
- Query optimization
- Connection pooling
- Pagination
- Batch writes
- Appropriate schema design
- Archiving old raw data

---

# 15. Important Database Indexes

Potential indexes include:

    city
    company
    opportunity type
    event date
    status
    discoveredAt
    lastCheckedAt

Composite indexes should be introduced based on actual query patterns.

Do not create indexes for every field without measuring.

---

# 16. Database Connection Pooling

As application instances increase, database connections can become a bottleneck.

The system should use controlled connection pools.

For example:

    Backend 1
       ↓
    Connection Pool

    Backend 2
       ↓
    Connection Pool

The total number of connections must remain within PostgreSQL capacity.

Adding more backend instances does not mean unlimited database connections.

---

# 17. Search Scalability

V1 search can use PostgreSQL.

As data grows, search requirements may become more complex.

Possible future search layer:

    PostgreSQL
        +
    Search Index

Potential technologies:

- OpenSearch
- Elasticsearch
- PostgreSQL full-text search

The decision should depend on:

- Dataset size
- Query complexity
- Ranking requirements
- Search latency

---

# 18. Caching

Redis can be introduced when repeated reads justify it.

Useful cache candidates:

- Popular searches
- City/technology normalization
- Source metadata
- Frequently accessed opportunities
- Search results with short TTL

Caching should not become the primary source of truth.

PostgreSQL remains authoritative.

---

# 19. Raw Document Storage

Raw HTML can consume significant storage.

At small scale it can remain in the database if appropriate.

At larger scale:

    Backend
       ↓
    Object Storage
       ↓
    Raw Documents

Possible storage:

- S3-compatible object storage
- Cloud object storage

The database can retain metadata and a reference to the stored document.

---

# 20. Deduplication at Scale

Deduplication can become expensive if every new opportunity is compared with every existing opportunity.

Avoid:

    New Job
       ↓
    Compare with every job

Instead use candidate matching.

First narrow candidates using:

    Company
    City
    Job Type
    Event Date

Then perform deeper similarity only on likely matches.

This keeps deduplication closer to:

    Candidate Matching
        ↓
    Detailed Comparison

rather than full-dataset comparison.

---

# 21. Idempotency at Scale

Retries are unavoidable in distributed processing.

A job may be processed more than once.

The system should therefore support idempotent processing.

Possible mechanisms:

- Unique database constraints
- Content hashes
- Opportunity identity keys
- Task IDs
- Processing state

Example:

    Same URL
        ↓
    Same normalized identity
        ↓
    Update existing record
        instead of
    Creating duplicate record

---

# 22. Retry Strategy

Not every failure should be retried.

Examples:

    Timeout
    Temporary network error
        → Retry

    404
        → Usually do not retry repeatedly

    CAPTCHA
        → Do not bypass

    Authentication required
        → Do not retry indefinitely

    Rate limit
        → Backoff

Retries should use controlled limits.

---

# 23. Source Reliability

Not every source is equally reliable.

The system should measure:

- Fetch success
- Extraction success
- Duplicate rate
- Expired rate
- Information completeness
- Historical freshness

This can later contribute to a source quality score.

---

# 24. Discovery Coverage Scaling

More sources do not automatically mean better results.

The goal is:

    Better Coverage
       +
    Better Relevance
       +
    Better Freshness

A source should be added when it provides meaningful incremental coverage.

This prevents unnecessary source complexity.

---

# 25. Scheduled Refresh

Future versions may periodically refresh stored opportunities.

Example:

    Active Opportunity
          ↓
    Scheduled Recheck
          ↓
    Source Fetch
          ↓
    Compare
          ↓
    Update

This allows the system to detect:

- Expired events
- Changed dates
- Changed venues
- Removed jobs
- Updated descriptions

---

# 26. Freshness Scaling

Not every opportunity needs the same refresh frequency.

Possible future strategy:

    Upcoming Event
        → Higher refresh priority

    Event far in future
        → Lower refresh priority

    Old / Aging
        → Lower priority

    Expired
        → No regular refresh

This reduces unnecessary network usage.

---

# 27. Priority-Based Processing

As workload grows, discovery tasks can be prioritized.

Example:

    HIGH
    User-triggered search

    MEDIUM
    Popular saved search

    LOW
    Background refresh

This ensures interactive user requests are not blocked by background work.

---

# 28. Multi-Region Future

International expansion may eventually require multiple regions.

Possible architecture:

    Global API
        ↓
    Regional Processing
        ├── India
        ├── US
        ├── Europe
        └── Other Regions

This is not required for the initial India-focused product.

---

# 29. Cost Scalability

The most important cost drivers may become:

- Search API usage
- HTTP fetching
- AI extraction
- Database storage
- Object storage
- Search infrastructure
- Compute

Therefore the platform should measure cost per discovered opportunity.

A useful future metric:

    Infrastructure Cost
    --------------------
    Valid Opportunities

This helps evaluate whether a new source or AI feature is economically useful.

---

# 30. AI Cost Control

If AI extraction is introduced, avoid:

    Every Page
        ↓
    AI

Prefer:

    Every Page
        ↓
    Cheap Deterministic Extraction
        ↓
    Confidence Check
        ↓
    AI Only If Needed

This can significantly reduce AI usage.

---

# 31. Scaling Architecture

A future scaled architecture may look like:

    Users
      ↓
    Load Balancer
      ↓
    API Instances
      ↓
    Discovery Queue
      ↓
    ┌───────────────────────────┐
    │ Worker Pool               │
    │                           │
    │ Discovery Workers         │
    │ Fetch Workers             │
    │ Extraction Workers        │
    │ Validation Workers        │
    └─────────────┬─────────────┘
                  ↓
          ┌───────────────┐
          │ Data Platform │
          │               │
          │ PostgreSQL    │
          │ Redis         │
          │ Search Index  │
          │ Object Store  │
          └───────────────┘

---

# 32. When to Introduce More Infrastructure

Infrastructure should be introduced when a measurable problem appears.

Examples:

    Problem:
    API latency

    Possible solution:
    Horizontal API scaling

    Problem:
    Long discovery tasks

    Possible solution:
    Worker queue

    Problem:
    Repeated database reads

    Possible solution:
    Redis

    Problem:
    Search query latency at large dataset

    Possible solution:
    Search index

    Problem:
    Raw storage growth

    Possible solution:
    Object storage

    Problem:
    Worker coordination at large scale

    Possible solution:
    Distributed messaging

The solution should follow the bottleneck.

---

# 33. What Not to Do

Do not introduce:

- Microservices
- Kafka
- Kubernetes
- Multiple databases
- Complex distributed systems

just because the product may become large.

Premature infrastructure increases:

- Development time
- Cost
- Operational complexity
- Debugging difficulty

Build the product first.

---

# 34. Scalability Path

The expected evolution is:

    Phase 1

    Spring Boot
    PostgreSQL
    Async Thread Pool

        ↓

    Phase 2

    Better indexing
    Redis if needed
    Search optimization

        ↓

    Phase 3

    Queue
    Multiple workers
    Horizontal API scaling

        ↓

    Phase 4

    Search Index
    Object Storage
    Advanced refresh system

        ↓

    Phase 5

    Selected service separation
    Multi-region infrastructure
    Advanced data platform

Each phase should be triggered by real product requirements.

---

# 35. Key Scalability Principle

The platform should scale the processing pipeline independently from the user-facing application.

The important separation is:

    User Traffic
        ≠
    Discovery Workload

User requests should remain fast even when the system is processing thousands of external pages.

That separation is one of the most important scalability decisions in the product.

---

# 36. Final Principle

The system should grow like this:

    Simple
       ↓
    Measured
       ↓
    Optimized
       ↓
    Distributed
       ↓
    Scaled

Not:

    Complex
       ↓
    Distributed
       ↓
    Expensive
       ↓
    Hard to maintain

The architecture should earn its complexity.
