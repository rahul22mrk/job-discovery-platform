# System Architecture

## 1. Purpose

The Job Discovery Platform is a web product for discovering relevant job opportunities from the public web.

V1 focuses on walk-in opportunities.

The system is designed so that the discovery engine can later support:

- Regular jobs
- Company-specific jobs
- Hiring drives
- Internships
- Saved searches
- Notifications
- Personalized discovery

The architecture starts as a modular monolith.

The backend is the primary development focus in V1 because the main technical challenge is discovering, processing, validating, and ranking real job opportunities from the public web.

The frontend is part of the product and repository from the beginning, but it will be implemented after the core backend pipeline and APIs are working reliably.

---

# 2. Repository Architecture

The project uses one GitHub repository.

    job-discovery-platform/
    ├── README.md
    ├── docs/
    │   ├── product/
    │   ├── requirements/
    │   └── architecture/
    ├── research/
    ├── roadmap/
    ├── backend/
    ├── frontend/
    ├── infrastructure/
    └── .github/

The two application areas are:

    backend/
    frontend/

The backend is developed first.

The frontend is implemented later using the stable backend APIs.

---

# 3. High-Level System

The complete product architecture is:

    ┌──────────────────────────┐
    │        FRONTEND          │
    │                          │
    │ Search                   │
    │ Results                  │
    │ Opportunity Details      │
    └────────────┬─────────────┘
                 │
                 │ HTTP / REST API
                 ▼
    ┌──────────────────────────┐
    │         BACKEND          │
    │                          │
    │ API                      │
    │ Discovery                │
    │ Search                   │
    │ Ranking                  │
    │ Processing               │
    └────────────┬─────────────┘
                 │
                 ▼
    ┌──────────────────────────┐
    │    DISCOVERY PIPELINE    │
    │                          │
    │ Query Generation         │
    │ Source Discovery         │
    │ URL Processing           │
    │ Page Fetching            │
    │ Extraction               │
    │ Normalization            │
    │ Validation               │
    │ Deduplication            │
    │ Freshness                │
    └────────────┬─────────────┘
                 │
                 ▼
    ┌──────────────────────────┐
    │       PostgreSQL         │
    │                          │
    │ Opportunities            │
    │ Companies                │
    │ Sources                  │
    │ Raw Documents            │
    │ Discovery Tasks          │
    └──────────────────────────┘

---

# 4. Architecture Priorities

The architecture prioritizes:

1. Correct discovery
2. Data quality
3. Source transparency
4. Freshness
5. Deduplication
6. Relevance
7. Reliability
8. Simplicity
9. Future scalability

Infrastructure complexity is intentionally kept low in V1.

---

# 5. Frontend

The frontend is a separate application inside the same repository.

    frontend/

It is not part of the backend codebase.

The frontend communicates with the backend through REST APIs.

## Responsibilities

- Search interface
- Search filters
- Discovery status
- Results display
- Opportunity details
- Source navigation
- Loading states
- Error states
- Empty states

The frontend should not contain discovery logic.

For example, the frontend should not:

- Search Google directly
- Fetch job websites
- Parse HTML
- Detect walk-ins
- Deduplicate opportunities
- Decide whether an opportunity is expired

These responsibilities belong to the backend.

---

# 6. Backend

The backend is the core of the product.

It is responsible for:

- Search requests
- Discovery tasks
- Query generation
- Source discovery
- Page fetching
- Extraction
- Normalization
- Validation
- Deduplication
- Freshness
- Ranking
- Persistence
- Search

The backend starts as a modular monolith.

---

# 7. Backend Logical Architecture

The backend can be divided into logical modules:

    api
    discovery
    extraction
    normalization
    validation
    deduplication
    freshness
    ranking
    search
    persistence
    common

These are modules inside one Spring Boot application.

They are not separate deployable services in V1.

---

# 8. API Layer

The API layer is the boundary between frontend and backend.

Responsibilities:

- Receive requests
- Validate requests
- Create discovery tasks
- Return task information
- Return search results
- Return opportunity details
- Return errors

Example:

    POST /api/v1/discovery/search

Future:

    GET /api/v1/discovery/search/{searchId}

    GET /api/v1/opportunities

    GET /api/v1/opportunities/{id}

---

# 9. Discovery Module

The Discovery module coordinates the discovery process.

Responsibilities:

- Create DiscoveryTask
- Build SearchContext
- Generate search queries
- Invoke source adapters
- Collect candidate URLs
- Start downstream processing

The Discovery module should orchestrate the pipeline.

It should not contain detailed HTML parsing or extraction logic.

---

# 10. Source Adapter Layer

External sources should be isolated behind interfaces.

Example:

    public interface JobSource {

        List<DiscoveredPage> discover(SearchContext context);

    }

Possible implementations:

    SearchEngineSource
    CompanyCareerSource
    JobBoardSource
    ApiJobSource
    RssSource

The rest of the application should depend on the interface rather than a specific provider.

---

# 11. Search Provider Independence

The architecture should avoid direct coupling between business logic and external provider SDKs.

Preferred:

    Discovery
        ↓
    JobSource
        ↓
    Provider Adapter
        ↓
    External Provider

This makes it possible to:

- Replace a provider
- Add another provider
- Disable a provider
- Compare providers
- Measure provider quality

without rewriting the discovery engine.

---

# 12. Fetcher

The Fetcher retrieves publicly accessible pages.

Responsibilities:

- HTTP requests
- Timeout
- Redirect handling
- Response validation
- Retry
- Backoff
- Content type handling
- Fetch metadata

The Fetcher should not contain business logic such as:

- Walk-in detection
- Job ranking
- Deduplication
- User search logic

---

# 13. External Web Access

The public web is an unreliable external dependency.

Pages may:

- Disappear
- Change structure
- Return errors
- Change content
- Block automated requests
- Become outdated

The architecture should expect failures.

One source or page failing must not stop the complete discovery task.

Only appropriately accessible public pages should be processed.

The system must not bypass:

- CAPTCHA
- Login
- Authentication
- Technical access controls
- Other restrictions intended to prevent automated access

Applicable crawling restrictions and source terms should be respected.

---

# 14. Extraction Module

The Extraction module converts raw documents into candidate job information.

Responsibilities:

- HTML parsing
- Text extraction
- JSON-LD extraction
- JobPosting structured-data extraction
- Source-specific parsing
- Pattern-based extraction
- Optional AI-assisted extraction

Extraction should use the simplest reliable method first.

Preferred order:

    Structured Data
        ↓
    Source-specific Parser
        ↓
    DOM Extraction
        ↓
    Text Patterns
        ↓
    AI-assisted Extraction

AI should not be mandatory for every page.

---

# 15. Walk-in Detection

Walk-in detection is a dedicated part of the extraction pipeline.

Possible signals:

- Walk-in interview
- Walk-in drive
- Walk-in recruitment
- Walk-in hiring
- Walkin
- Interview date
- Interview time
- Interview venue
- Recruitment drive

Possible classification:

    WALK_IN
    NOT_WALK_IN
    UNCERTAIN

The system should use multiple signals instead of relying on a single keyword.

---

# 16. Normalization Module

The Normalization module converts source-specific representations into a common format.

Examples:

    Bangalore → Bengaluru

    3-5 years → min=3, max=5

    3+ years → min=3

Technology aliases can also be normalized.

Example:

    Core Java
    Java Developer
    Java Backend
    Java Engineer

can be mapped into a common representation for matching.

---

# 17. Validation Module

The Validation module checks whether extracted information is usable.

Validation may include:

- Company validation
- Job title validation
- Location validation
- Event date validation
- Experience validation
- Walk-in evidence
- Source URL validation
- Expiration checks

The system should reject unreliable information rather than guess.

---

# 18. Deduplication Module

The same opportunity can appear on multiple websites.

Example:

    Company A
    Java Developer
    Bengaluru
    15 September

may appear on:

    Company Website
    Job Board A
    Job Board B
    Aggregator

These should normally represent one logical opportunity.

Initial duplicate signals:

- Company
- Normalized job title
- City
- Event date
- Venue
- Application URL

More advanced similarity can be added later.

---

# 19. Source Model

An opportunity can have multiple sources.

Example:

    Opportunity
       ├── Company Website
       ├── Job Board
       └── Aggregator

The system should maintain this relationship instead of storing only one source.

This provides:

- Source transparency
- Better duplicate detection
- Trust signals
- Source quality measurement

---

# 20. Freshness Module

Walk-in opportunities are time-sensitive.

The system should track:

- firstSeenAt
- discoveredAt
- lastCheckedAt
- eventDate
- status

Initial states:

    ACTIVE
    AGING
    EXPIRED

Basic rule:

    eventDate < current date
        ↓
    EXPIRED

Future states may include:

    CANCELLED
    RESCHEDULED

---

# 21. Ranking Module

The Ranking module determines result order.

Initial signals:

1. Technology match
2. Role match
3. City match
4. Experience match
5. Event date relevance
6. Freshness
7. Source quality
8. Information completeness

The first version should use a simple scoring model.

Ranking should be measurable and improvable.

---

# 22. Search Module

The Search module retrieves stored opportunities based on user intent.

Initial filters:

- City
- Keyword
- Experience
- Date range
- Opportunity type
- Company when required

PostgreSQL is sufficient for the initial search requirements.

A dedicated search engine can be introduced later if search complexity or dataset size requires it.

---

# 23. Persistence Layer

PostgreSQL is the initial source of truth.

Potential tables:

    companies
    job_postings
    job_sources
    job_raw_documents
    job_discovery_tasks
    job_skills
    job_posting_skills

Future:

    users
    saved_searches
    notifications
    applications

---

# 24. Raw Data Architecture

Raw source data should be separated from normalized opportunity data.

Flow:

    External Page
        ↓
    Raw Document
        ↓
    Extraction
        ↓
    Normalized JobPosting

This allows the system to:

- Reprocess old documents
- Improve extraction logic
- Debug incorrect results
- Detect page changes

---

# 25. Asynchronous Processing

Discovery can involve many external requests and should not run entirely inside a normal HTTP request.

Flow:

    Frontend
       ↓
    Backend API
       ↓
    Create Discovery Task
       ↓
    Async Processing
       ↓
    Discovery Pipeline
       ↓
    PostgreSQL
       ↓
    Backend API
       ↓
    Frontend

The API should return quickly after creating the discovery task.

Example:

    {
      "searchId": "abc123",
      "status": "PROCESSING"
    }

---

# 26. V1 Async Strategy

V1 can use:

    Spring Async
        +
    Controlled Thread Pool

This keeps the architecture simple.

A distributed queue such as Kafka can be introduced later if actual workload requires:

- Multiple workers
- Durable task queues
- High processing volume
- More advanced retry handling
- Distributed processing

---

# 27. Error Isolation

Failures should be isolated.

Example:

    Source A
        ↓
    Failed

    Source B
        ↓
    Success

    Source C
        ↓
    Success

The discovery task should continue.

Similarly:

    Page A
        ↓
    Extraction Failed

    Page B
        ↓
    Extraction Successful

Page A should not prevent Page B from being processed.

---

# 28. Idempotency

The pipeline should be safe to retry.

The same page or opportunity may be processed more than once.

Possible mechanisms:

- URL normalization
- Content hashes
- Unique database constraints
- Opportunity identity
- Discovery task IDs

Example:

    Same Opportunity
        ↓
    Same logical identity
        ↓
    Update existing record

instead of:

    Create another duplicate

---

# 29. Observability

The system should produce useful logs and metrics.

Track:

- Discovery task
- Source
- URL
- Fetch result
- Extraction result
- Validation result
- Duplicate result
- Processing time
- Database operation

Important identifiers:

    searchId
    discoveryTaskId
    sourceId
    documentId
    opportunityId

These identifiers make debugging easier.

---

# 30. Security

Protect:

- Database credentials
- API keys
- Search provider credentials
- Internal configuration

Secrets should never be committed to Git.

Use:

- Environment variables
- Secret management
- Secure configuration

Authentication and authorization can be added when user accounts are introduced.

---

# 31. Deployment Architecture — Initial

The first deployable architecture can remain simple.

    Internet
       ↓
    Frontend
       ↓
    Backend
       ↓
    PostgreSQL

The backend remains a single deployable application.

The frontend remains a separate application.

No microservices are required initially.

---

# 32. Future Deployment Architecture

As workload increases:

    Internet
       ↓
    Load Balancer
       ↓
    ┌───────────────┐
    │ API Instance 1│
    │ API Instance 2│
    │ API Instance 3│
    └───────┬───────┘
            ↓
       Job Queue
            ↓
    ┌─────────────────────┐
    │ Processing Workers  │
    │                     │
    │ Discovery           │
    │ Fetching            │
    │ Extraction          │
    │ Validation          │
    └──────────┬──────────┘
               ↓
    ┌─────────────────────┐
    │ Data Layer          │
    │                     │
    │ PostgreSQL          │
    │ Redis               │
    │ Search Index        │
    │ Object Storage      │
    └─────────────────────┘

This is a future evolution, not a V1 requirement.

---

# 33. Why Modular Monolith First

A modular monolith provides:

- Faster development
- Easier local testing
- Simple deployment
- Lower infrastructure cost
- Easier debugging
- Clear internal boundaries

Microservices would introduce additional complexity such as:

- Network communication
- Service discovery
- Distributed tracing
- Multiple deployments
- More failure modes
- More operational work

That complexity is not justified at the beginning.

---

# 34. Architecture Evolution

The system can evolve gradually.

Stage 1:

    Modular Monolith
    PostgreSQL
    Async Processing

Stage 2:

    Modular Monolith
    PostgreSQL
    Redis if required
    Search optimization

Stage 3:

    Queue
    Multiple Workers
    Horizontal API Scaling

Stage 4:

    Search Index
    Object Storage
    Advanced Refresh Processing

Stage 5:

    Selected Services
    Distributed Infrastructure
    Multi-region Deployment

The transition should be driven by actual bottlenecks.

---

# 35. Main Architectural Boundary

The most important boundary is:

    External Web
        ↓
    Source Adapters
        ↓
    Discovery Pipeline
        ↓
    Domain Model
        ↓
    Database
        ↓
    API
        ↓
    Frontend

The external web is unpredictable.

The internal domain model should remain stable.

---

# 36. Core Architecture Principle

The system should optimize for:

    Relevance
        >
    Freshness
        >
    Trust
        >
    Quantity

The goal is not to collect the maximum number of pages.

The goal is to turn public web data into useful, trustworthy job opportunities.

---

# 37. Final V1 Architecture

    ┌───────────────────────┐
    │       FRONTEND        │
    │                       │
    │ Search               │
    │ Results              │
    │ Details              │
    └──────────┬────────────┘
               │
               ▼
    ┌───────────────────────┐
    │     SPRING BOOT       │
    │   MODULAR MONOLITH    │
    │                       │
    │ API                   │
    │ Discovery             │
    │ Extraction            │
    │ Normalization         │
    │ Validation            │
    │ Deduplication         │
    │ Freshness             │
    │ Ranking               │
    │ Search                │
    └──────────┬────────────┘
               │
               ▼
    ┌───────────────────────┐
    │      POSTGRESQL       │
    │                       │
    │ Opportunities         │
    │ Sources               │
    │ Raw Documents         │
    │ Discovery Tasks       │
    └───────────────────────┘

The frontend exists in the repository from Day 1, but backend discovery and data processing are the first implementation priority.

Frontend development begins after the backend has a reliable discovery pipeline and stable APIs.
:::

**Ab next `docs/architecture/scalability.md` ko bhi isi corrected approach ke according update karenge.**
