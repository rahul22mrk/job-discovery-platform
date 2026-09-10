# System Architecture

## 1. Purpose

The Job Discovery Platform is designed as a source-independent job discovery system.

The initial product focuses on walk-in opportunities.

The architecture should allow the platform to later support:

- Regular jobs
- Company-specific jobs
- Hiring drives
- Internships
- Saved searches
- Notifications
- Personalized job discovery

The first implementation should use a modular monolith.

Microservices should not be introduced until real scale or operational requirements justify them.

---

# 2. Architecture Goals

The architecture should provide:

- Simple development
- Clear module boundaries
- Source independence
- Reliable processing
- Data quality
- Freshness
- Deduplication
- Easy testing
- Easy future expansion
- Controlled infrastructure complexity

The system should be built around product requirements rather than technology choices.

---

# 3. High-Level Architecture

    ┌───────────────────────┐
    │       Frontend        │
    │   Search / Results    │
    └───────────┬───────────┘
                │
                ▼
    ┌───────────────────────┐
    │       REST API        │
    │ Search / Opportunities │
    └───────────┬───────────┘
                │
                ▼
    ┌─────────────────────────────────┐
    │        Application Layer        │
    │                                 │
    │ Discovery                       │
    │ Search                          │
    │ Ranking                         │
    │                                 │
    └──────────────┬──────────────────┘
                   │
                   ▼
    ┌─────────────────────────────────┐
    │         Processing Layer        │
    │                                 │
    │ Query Generation                │
    │ Source Discovery                │
    │ Fetching                        │
    │ Extraction                      │
    │ Normalization                   │
    │ Validation                      │
    │ Deduplication                   │
    │ Freshness                       │
    └──────────────┬──────────────────┘
                   │
                   ▼
    ┌─────────────────────────────────┐
    │          Data Layer             │
    │                                 │
    │ PostgreSQL                      │
    │ Raw Documents                   │
    │ Opportunities                   │
    │ Sources                         │
    │ Discovery Tasks                 │
    └─────────────────────────────────┘

---

# 4. Main Components

The system consists of the following logical components:

    API
    Discovery
    Source Adapters
    Fetcher
    Extraction
    Normalization
    Validation
    Deduplication
    Freshness
    Ranking
    Search
    Persistence
    Common

These are logical modules.

They do not need to be separate deployable services in V1.

---

# 5. API Layer

The API layer handles requests from the frontend or other clients.

Responsibilities:

- Request validation
- Authentication in the future
- API response formatting
- Starting discovery tasks
- Returning search results
- Returning task status

Example endpoint:

    POST /api/v1/discovery/search

Future:

    GET /api/v1/discovery/search/{searchId}

    GET /api/v1/opportunities

    GET /api/v1/opportunities/{id}

---

# 6. Discovery Module

The Discovery module coordinates the discovery process.

Responsibilities:

- Create discovery task
- Build SearchContext
- Generate queries
- Invoke source adapters
- Collect candidate URLs
- Start processing

The Discovery module should orchestrate the workflow but should not contain detailed HTML parsing logic.

---

# 7. Source Adapter Layer

Different sources should be hidden behind a common interface.

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

This prevents the rest of the application from becoming dependent on one provider.

---

# 8. Search Provider Independence

The architecture should avoid:

    Business Logic
        ↓
    Provider-specific API

Instead:

    Business Logic
        ↓
    JobSource
        ↓
    Provider Adapter

This allows the product to replace or add providers without rewriting the discovery pipeline.

---

# 9. Fetcher

The Fetcher is responsible only for retrieving public documents.

Responsibilities:

- HTTP requests
- Timeout
- Redirect handling
- Retry
- Backoff
- Response validation
- Fetch metadata

It should not contain:

- Walk-in detection
- Job extraction
- Ranking
- Business rules

---

# 10. Extraction Module

The Extraction module converts raw documents into candidate job information.

Responsibilities:

- HTML parsing
- Text extraction
- JSON-LD extraction
- JobPosting extraction
- Source-specific extraction
- Pattern-based extraction
- Optional AI-assisted extraction

Interface:

    JobExtractor

Possible implementations:

    StructuredDataExtractor
    HtmlJobExtractor
    WalkInExtractor
    SourceSpecificExtractor

---

# 11. Normalization Module

Normalization creates a common representation.

Responsibilities:

- City normalization
- Company normalization
- Job title normalization
- Skill normalization
- Experience normalization
- Date normalization
- Time normalization

Example:

    Bangalore
    Bengaluru
    Bangalore Urban

can be mapped to a canonical representation where appropriate.

---

# 12. Validation Module

Validation ensures that extracted data is usable.

Responsibilities:

- Required field validation
- Date validation
- Location validation
- Walk-in validation
- Experience validation
- Source validation
- Expiry validation

The module should reject unreliable information instead of guessing.

---

# 13. Deduplication Module

The Deduplication module identifies opportunities representing the same real-world event/job.

Initial matching:

    Company
    +
    Job Title
    +
    City
    +
    Event Date

Additional signals:

- Venue
- Application URL
- Description similarity
- Source relationship

The module should eventually support confidence-based matching.

---

# 14. Freshness Module

Freshness determines whether an opportunity should remain active.

Responsibilities:

- Event date checks
- Last checked timestamp
- Aging calculation
- Expiration
- Future change detection

Initial states:

    ACTIVE
    AGING
    EXPIRED

---

# 15. Ranking Module

Ranking determines the order in which results are shown.

Example scoring signals:

    Technology Match
    Role Match
    City Match
    Experience Match
    Date Relevance
    Freshness
    Source Quality
    Completeness

The ranking implementation should initially remain simple.

---

# 16. Search Module

The Search module handles queries against stored opportunities.

Responsibilities:

- Filter by city
- Filter by keyword
- Filter by experience
- Filter by date
- Filter by opportunity type
- Sort by relevance
- Return active opportunities

PostgreSQL can handle the initial search requirements.

A dedicated search engine should only be introduced when PostgreSQL is no longer sufficient.

---

# 17. Persistence Layer

PostgreSQL is the initial primary database.

Potential tables:

    companies
    job_postings
    job_sources
    job_raw_documents
    job_discovery_tasks
    job_skills
    job_posting_skills

Future tables:

    users
    saved_searches
    notifications
    applications

---

# 18. Domain Model

The domain should be generic enough for future opportunity types.

Example:

    JobOpportunityType

    WALK_IN
    JOB
    HIRING_DRIVE

V1 primarily uses:

    WALK_IN

The architecture should not create a completely separate system for every opportunity type.

---

# 19. Opportunity Model

A JobPosting should contain concepts such as:

- Company
- Title
- Description
- Location
- Opportunity type
- Experience
- Skills
- Event information
- Source
- Freshness
- Status
- Timestamps

The model should distinguish between:

    Job

and:

    Source information about the Job

One logical opportunity may have multiple sources.

---

# 20. Raw Data Architecture

Raw source data should be separated from normalized data.

    External Source
          ↓
    Raw Document
          ↓
    Extraction
          ↓
    Normalized Opportunity

This allows extraction logic to evolve without losing the original source information.

---

# 21. Asynchronous Processing

Discovery can take longer than a normal API request.

Therefore:

    User
      ↓
    API
      ↓
    Discovery Task
      ↓
    Async Processing
      ↓
    Database
      ↓
    Results

The API should return quickly after creating the discovery task.

Example:

    {
      "searchId": "abc123",
      "status": "PROCESSING"
    }

---

# 22. V1 Async Strategy

V1 should use a simple controlled asynchronous mechanism.

For example:

    Spring Async
    +
    Thread Pool

This is sufficient for the initial product.

A distributed queue such as Kafka can be introduced later if:

- Work volume becomes large
- Multiple workers are required
- Retry requirements become complex
- Event-driven processing provides measurable value

---

# 23. Transaction Boundaries

Database transactions should be kept around meaningful units of work.

Examples:

    Save normalized opportunity

    Update discovery task status

    Save source relationship

Large network operations should not run inside database transactions.

Do not keep database transactions open while fetching external pages.

---

# 24. Error Isolation

External sources are unreliable.

Architecture should ensure:

    Source A fails
         ↓
    Source B continues
         ↓
    Source C continues

Similarly:

    Page A extraction fails
         ↓
    Page B continues

Failure should be isolated at the smallest useful unit.

---

# 25. Observability

The system should provide logs and metrics for:

- Discovery tasks
- Source requests
- Fetch failures
- Extraction failures
- Validation failures
- Duplicate detection
- Processing time
- Database operations

Important identifiers:

    searchId
    discoveryTaskId
    sourceId
    documentId
    opportunityId

These identifiers make debugging easier.

---

# 26. Security

The system should protect:

- Database credentials
- API keys
- Search provider credentials
- Internal endpoints

Secrets should not be stored in source code.

Use:

- Environment variables
- Secret management
- Secure configuration

Future authentication/authorization will be added when user accounts are introduced.

---

# 27. External Dependency Isolation

External services should be accessed through adapters.

Examples:

    SearchProviderAdapter
    JobBoardAdapter
    CompanyCareerAdapter

The core domain should not depend directly on SDK-specific classes.

This makes external provider replacement easier.

---

# 28. Deployment Architecture — V1

Initial deployment can remain simple:

    Internet
       ↓
    Frontend
       ↓
    Backend
       ↓
    PostgreSQL

The backend can contain all logical modules in one application.

Docker can be introduced for reproducible deployment.

Cloud deployment can be added after the local product works reliably.

---

# 29. Future Deployment Architecture

At higher scale:

    Load Balancer
         ↓
    API Instances
         ↓
    Job Queue
         ↓
    Worker Instances
         ↓
    Discovery Workers
    Fetch Workers
    Extraction Workers
         ↓
    PostgreSQL
         +
    Cache
         +
    Search Index
         +
    Object Storage

The logical modules remain similar even if they later become separate services.

---

# 30. Why Modular Monolith First

A modular monolith provides:

- Faster development
- Easier local testing
- Simpler deployment
- Lower infrastructure cost
- Easier debugging
- Clear module boundaries

Microservices introduce:

- Network communication
- Service discovery
- Distributed tracing
- More deployments
- More monitoring
- More failure modes

These costs are not justified for the first version.

---

# 31. Architecture Evolution

The architecture should evolve approximately like this:

    Stage 1

    Modular Monolith
    PostgreSQL
    Async Thread Pool

        ↓

    Stage 2

    Modular Monolith
    PostgreSQL
    Redis
    Search Index

        ↓

    Stage 3

    Queue
    Multiple Workers
    Horizontal Scaling

        ↓

    Stage 4

    Selected Modules as Services

The transition should happen only when measurements justify it.

---

# 32. Key Architecture Principle

The most important architectural boundary is:

    External Web
          ↓
    Source Adapters
          ↓
    Discovery Pipeline
          ↓
    Domain Model
          ↓
    Search / Product

The external web is unpredictable.

The internal domain model should remain stable.

---

# 33. Final Architecture

V1:

    Frontend
       ↓
    REST API
       ↓
    Modular Monolith
       ├── Discovery
       ├── Source Adapters
       ├── Fetcher
       ├── Extraction
       ├── Normalization
       ├── Validation
       ├── Deduplication
       ├── Freshness
       ├── Ranking
       └── Search
       ↓
    PostgreSQL

Future:

    Frontend
       ↓
    API Layer
       ↓
    Queue / Workers
       ├── Discovery
       ├── Fetching
       ├── Extraction
       ├── Validation
       └── Processing
       ↓
    Data Platform
       ├── PostgreSQL
       ├── Redis
       ├── Search Index
       └── Object Storage

The architecture should scale by separating the bottlenecks, not by splitting everything into microservices from day one.
