# Backend Architecture

## 1. Purpose

The backend is the core of the Job Discovery Platform.

Its primary responsibility is to discover, process, validate, store, and serve useful job opportunities from the public web.

V1 focuses on walk-in opportunities.

The backend should be designed as a modular monolith.

The modules should have clear responsibilities and dependencies so that the system can evolve without becoming tightly coupled.

---

# 2. Backend Technology

Initial backend stack:

    Java
    Spring Boot
    Spring Web
    Spring Data JPA
    Hibernate
    PostgreSQL
    Maven

Additional technologies should be introduced only when required.

Possible future technologies:

    Redis
    Kafka
    OpenSearch / Elasticsearch
    Docker
    AWS

These are not required for the initial backend.

---

# 3. Backend Architecture Style

The backend uses a modular monolith.

    One Spring Boot Application
             │
             ├── API
             ├── Discovery
             ├── Extraction
             ├── Normalization
             ├── Validation
             ├── Deduplication
             ├── Freshness
             ├── Ranking
             ├── Search
             └── Persistence

All modules run inside the same application.

The modules should communicate through well-defined interfaces and domain objects.

---

# 4. Main Backend Flow

    API Request
        ↓
    Discovery Service
        ↓
    Query Generator
        ↓
    Job Sources
        ↓
    URL Processing
        ↓
    Page Fetcher
        ↓
    Extraction
        ↓
    Normalization
        ↓
    Validation
        ↓
    Deduplication
        ↓
    Freshness
        ↓
    Persistence
        ↓
    Ranking / Search
        ↓
    API Response

---

# 5. API Module

## Responsibility

The API module exposes backend functionality to the frontend.

Responsibilities:

- Request handling
- Request validation
- Response mapping
- Exception handling
- API versioning
- HTTP status handling

Example endpoints:

    POST /api/v1/discovery/search

    GET /api/v1/discovery/search/{searchId}

    GET /api/v1/opportunities

    GET /api/v1/opportunities/{id}

The API module should not contain discovery logic.

---

# 6. Discovery Module

## Responsibility

The Discovery module coordinates the complete discovery process.

Responsibilities:

- Create discovery task
- Build SearchContext
- Generate queries
- Invoke JobSource implementations
- Collect candidate URLs
- Start processing
- Track task progress

Main concepts:

    SearchContext
    DiscoveryTask
    DiscoveredPage
    JobSource

The Discovery module acts as an orchestrator.

---

# 7. SearchContext

SearchContext represents user search intent internally.

Possible fields:

    city
    country
    company
    keyword
    skills
    type
    experienceMin
    experienceMax
    fromDate
    toDate

The SearchContext should be independent of:

- HTTP
- Database
- Search provider SDKs

This keeps the domain model reusable.

---

# 8. Query Generation

The Query Generator converts SearchContext into discovery queries.

Example:

    SearchContext:

    City = Bengaluru
    Keyword = Java
    Type = WALK_IN

Possible queries:

    "walk-in interview" "Bengaluru" "Java"
    "walk-in drive" "Bengaluru" "Java"
    "walkin interview" "Bengaluru" "Java"
    "Java developer" "walk-in" "Bengaluru"

The query generator should handle:

- Aliases
- Synonyms
- City normalization
- Role variations
- Walk-in terminology

---

# 9. JobSource Interface

External discovery sources should use a common interface.

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

The Discovery module should depend on the interface.

It should not depend on a concrete provider.

---

# 10. Source Adapter Responsibility

A source adapter should only be responsible for communicating with its external source.

It may handle:

- Provider request
- Authentication if legitimately required
- Provider response
- Provider-specific parsing
- Provider errors
- Provider rate limits

It should return the application's internal representation.

It should not perform:

- Final job validation
- Deduplication
- Ranking
- User-specific filtering

---

# 11. Fetching Module

## Responsibility

Fetch publicly accessible web pages.

Responsibilities:

- HTTP requests
- Timeouts
- Redirect handling
- Content type
- Retry
- Backoff
- Fetch metadata

Input:

    URL

Output:

    RawDocument

The Fetcher should be independent from extraction.

---

# 12. External Access Rules

The backend should process only appropriately accessible public content.

It must not bypass:

- CAPTCHA
- Authentication
- Login requirements
- Technical access controls

Applicable crawling restrictions and source terms should be respected.

If a page cannot be appropriately accessed:

    Record Failure
        ↓
    Continue Pipeline

One inaccessible source must not stop the discovery task.

---

# 13. Extraction Module

## Responsibility

Convert RawDocument into candidate job information.

Responsibilities:

- HTML parsing
- Text extraction
- JSON-LD extraction
- JobPosting extraction
- Source-specific extraction
- Pattern extraction
- Optional AI-assisted extraction

Potential interfaces:

    JobExtractor

Example:

    public interface JobExtractor {

        Optional<JobPostingCandidate> extract(
            RawDocument document
        );

    }

---

# 14. Walk-in Detection

Walk-in detection belongs close to the extraction layer.

The detector evaluates signals such as:

- Walk-in interview
- Walk-in drive
- Walk-in recruitment
- Walk-in hiring
- Interview date
- Interview time
- Interview venue

Possible output:

    WALK_IN
    NOT_WALK_IN
    UNCERTAIN

The detector should not rely only on a single keyword.

---

# 15. Normalization Module

## Responsibility

Convert extracted values into a common internal format.

Examples:

    Bangalore → Bengaluru

    3-5 years → min=3, max=5

    3+ years → min=3

Normalize:

- Company
- City
- Job title
- Skills
- Experience
- Dates
- Times

Normalization should be deterministic where possible.

---

# 16. Validation Module

## Responsibility

Determine whether extracted information is acceptable.

Validate:

- Company
- Job title
- Location
- Source URL
- Walk-in evidence
- Event date
- Experience
- Data format
- Expiration

Validation should reject unreliable information rather than guessing.

---

# 17. Deduplication Module

## Responsibility

Identify multiple records representing the same opportunity.

Initial identity signals:

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

The deduplication module should return either:

    New Opportunity

or:

    Existing Opportunity

or:

    Possible Duplicate

---

# 18. Freshness Module

## Responsibility

Determine whether an opportunity is still relevant.

Track:

    firstSeenAt
    discoveredAt
    lastCheckedAt
    eventDate
    status

Initial statuses:

    ACTIVE
    AGING
    EXPIRED

The module should prevent expired walk-ins from appearing as active opportunities.

---

# 19. Ranking Module

## Responsibility

Calculate how useful an opportunity is for a search.

Initial ranking signals:

- Technology match
- Role match
- City match
- Experience match
- Event date
- Freshness
- Source quality
- Completeness

The ranking algorithm should remain simple and measurable initially.

---

# 20. Search Module

## Responsibility

Search stored opportunities.

Initial filters:

- City
- Keyword
- Experience
- Date
- Company
- Opportunity type

PostgreSQL is sufficient initially.

A search engine can be added later if required.

---

# 21. Persistence Module

## Responsibility

Handle database operations.

Initial database:

    PostgreSQL

Potential entities:

    Company
    JobPosting
    JobSource
    RawDocument
    DiscoveryTask
    Skill

The persistence layer should hide database implementation details from business logic.

---

# 22. Domain Layer

The domain layer contains concepts that represent the product.

Important domain concepts:

    JobPosting
    Company
    JobSource
    DiscoveryTask
    SearchContext
    DiscoveredPage
    RawDocument
    Skill
    JobOpportunityType

Business rules should be expressed around these concepts.

---

# 23. DTO Layer

API DTOs should remain separate from domain entities.

Example:

    JobSearchRequest
    JobSearchResponse
    OpportunityResponse

Do not expose JPA entities directly from controllers.

Flow:

    API DTO
       ↓
    Service
       ↓
    Domain
       ↓
    Repository
       ↓
    Database

And:

    Database
       ↓
    Domain
       ↓
    Service
       ↓
    Response DTO
       ↓
    API

---

# 24. Repository Layer

Repositories are responsible for database access.

Examples:

    CompanyRepository
    JobPostingRepository
    JobSourceRepository
    RawDocumentRepository
    DiscoveryTaskRepository

Business logic should remain outside repository classes.

---

# 25. Service Layer

Services coordinate business operations.

Examples:

    DiscoveryService
    ExtractionService
    ValidationService
    DeduplicationService
    FreshnessService
    RankingService
    SearchService

Services should depend on interfaces where external behavior is involved.

---

# 26. Dependency Direction

The preferred dependency direction is:

    API
      ↓
    Application / Services
      ↓
    Domain
      ↓
    Infrastructure

External systems should be accessed through interfaces.

For example:

    DiscoveryService
          ↓
    JobSource
          ↓
    SearchEngineSource
          ↓
    External Provider

The core application should not depend directly on provider-specific implementations.

---

# 27. Suggested Package Structure

The initial package structure can remain simple:

    com.jobdiscovery

    ├── api
    │   ├── controller
    │   ├── dto
    │   └── exception
    │
    ├── discovery
    │   ├── service
    │   ├── model
    │   └── source
    │
    ├── extraction
    │   ├── service
    │   ├── parser
    │   └── detector
    │
    ├── normalization
    │
    ├── validation
    │
    ├── deduplication
    │
    ├── freshness
    │
    ├── ranking
    │
    ├── search
    │
    ├── persistence
    │   ├── entity
    │   └── repository
    │
    └── common

This is a starting structure.

It should evolve as the implementation grows.

---

# 28. Avoid Premature Packages

Do not create dozens of packages before there is code that needs them.

For example, do not create separate modules for:

    Kafka
    Redis
    AI
    Kubernetes
    Notifications
    Payments

until those features actually exist.

The package structure should grow with the product.

---

# 29. Async Processing

Discovery should be asynchronous.

Flow:

    Controller
        ↓
    DiscoveryService
        ↓
    Create DiscoveryTask
        ↓
    Async Worker
        ↓
    Discovery Pipeline

V1 can use:

    Spring Async
    +
    Thread Pool

Future:

    Queue
    +
    Distributed Workers

---

# 30. Transaction Boundaries

Database transactions should cover database operations.

Avoid keeping a database transaction open during external HTTP calls.

Bad:

    Start Transaction
        ↓
    Fetch Website
        ↓
    Parse Website
        ↓
    Commit

Preferred:

    Fetch Website
        ↓
    Process
        ↓
    Validate
        ↓
    Start DB Transaction
        ↓
    Persist
        ↓
    Commit

This reduces long-running database transactions.

---

# 31. Error Handling

The backend should use centralized error handling.

API errors should have a consistent format.

Example:

    {
      "timestamp": "...",
      "status": 400,
      "error": "BAD_REQUEST",
      "message": "Invalid search request",
      "path": "/api/v1/discovery/search"
    }

Internal pipeline failures should be tracked separately.

External source failures should not necessarily become user-facing API errors.

---

# 32. Logging

Useful logs should include:

    discoveryTaskId
    searchId
    sourceId
    documentId
    opportunityId

Example events:

    Discovery task started
    Query generated
    Source search completed
    URL discovered
    Page fetched
    Extraction completed
    Validation failed
    Duplicate detected
    Opportunity stored
    Discovery task completed

Avoid logging sensitive credentials or secrets.

---

# 33. Testing Strategy

Testing should exist at multiple levels.

## Unit Tests

Test:

- Query generation
- Normalization
- Walk-in detection
- Validation
- Deduplication
- Ranking

## Integration Tests

Test:

- PostgreSQL
- Repositories
- API
- Discovery workflow

## End-to-End Tests

Test:

    Search Request
        ↓
    Discovery
        ↓
    Processing
        ↓
    Database
        ↓
    Results API

Real public web testing should be controlled and separated from normal automated tests.

---

# 34. Configuration

External configuration should include:

- Database configuration
- Provider credentials
- HTTP timeouts
- Retry settings
- Thread pool settings
- Feature flags

Secrets must not be committed to Git.

Use environment variables or appropriate secret management.

---

# 35. Backend Evolution

Initial:

    Modular Monolith
        +
    PostgreSQL
        +
    Async Thread Pool

Future:

    Modular Monolith
        +
    Queue
        +
    Multiple Workers
        +
    Redis
        +
    Search Index

Only later, if justified:

    Selected Microservices

The domain boundaries should remain stable throughout this evolution.

---

# 36. Core Backend Principle

The backend should separate:

    Discovery
        ↓
    Processing
        ↓
    Domain
        ↓
    Persistence
        ↓
    API

External web complexity should remain at the source/fetching/extraction boundaries.

The core domain should remain clean and predictable.

---

# 37. Final Backend Architecture

    ┌───────────────────────────────┐
    │           REST API            │
    └───────────────┬───────────────┘
                    ↓
    ┌───────────────────────────────┐
    │       Application Layer        │
    │                               │
    │ Discovery                     │
    │ Search                        │
    │ Ranking                       │
    └───────────────┬───────────────┘
                    ↓
    ┌───────────────────────────────┐
    │       Processing Layer        │
    │                               │
    │ Source Adapters               │
    │ Fetching                      │
    │ Extraction                    │
    │ Normalization                 │
    │ Validation                    │
    │ Deduplication                 │
    │ Freshness                     │
    └───────────────┬───────────────┘
                    ↓
    ┌───────────────────────────────┐
    │          Domain Layer         │
    │                               │
    │ JobPosting                    │
    │ Company                       │
    │ Source                        │
    │ DiscoveryTask                 │
    │ SearchContext                 │
    └───────────────┬───────────────┘
                    ↓
    ┌───────────────────────────────┐
    │       Persistence Layer       │
    │                               │
    │ PostgreSQL                    │
    └───────────────────────────────┘

The backend is the core of V1.

The architecture should remain simple enough to develop quickly while keeping clear boundaries for future scale.
