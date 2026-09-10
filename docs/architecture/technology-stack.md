# Technology Stack

## Purpose

This document defines the technology choices for V1 of the Job Discovery Platform.

The goal is not to use as many technologies as possible.

The goal is to use a small, reliable stack that is sufficient to build, test, and run the first real version of the product.

---

## 1. Backend

### Language

Java

### Java Version

Java 21

Java 21 will be the primary backend development version.

Reasons:

- Long-term support release
- Modern Java features
- Good Spring Boot support
- Suitable for production backend development

---

## 2. Backend Framework

Spring Boot

Spring Boot will be used for:

- REST APIs
- Dependency injection
- Configuration
- Validation
- Async processing
- Exception handling
- Application lifecycle
- Integration with persistence

The backend will initially be a modular monolith.

Microservices are not required for V1.

---

## 3. Web Layer

Spring Web

Spring MVC will be used to build REST APIs.

Responsibilities:

- Request handling
- Request validation
- Response serialization
- HTTP status handling
- REST controllers

Controllers should remain thin.

Business logic belongs in service/domain modules.

---

## 4. API Style

REST

Initial API base path:

/api/v1

Core V1 APIs:

POST /api/v1/discovery/search

GET /api/v1/discovery/search/{searchId}

GET /api/v1/opportunities

GET /api/v1/opportunities/{id}

GET /actuator/health

The API surface should remain small until additional product features require more endpoints.

---

## 5. Database

PostgreSQL

PostgreSQL will be the primary source of truth.

It will store:

- Companies
- Job opportunities
- Sources
- Opportunity-source relationships
- Skills
- Discovery tasks
- Freshness state
- Normalized job information

The initial application does not require a separate search database.

---

## 6. Persistence

Spring Data JPA

Hibernate

JPA/Hibernate will be used for:

- Entity mapping
- Repository access
- Transactions
- Relationship management
- Database persistence

Database entities should not be exposed directly through REST APIs.

Use DTOs at the API boundary.

---

## 7. Database Migrations

A versioned database migration tool should be used.

The selected migration tool can be:

Flyway

or another equivalent migration solution if a project-level decision changes.

The important rule is:

Database schema changes must be version-controlled and reproducible.

Do not depend on manually modifying production database schemas.

---

## 8. Build Tool

Maven

Maven will be used for:

- Dependency management
- Compilation
- Testing
- Packaging
- Spring Boot application build

The project should use a standard Maven structure.

---

## 9. Validation

Spring Boot validation with Jakarta Validation.

Use annotations such as:

- @NotNull
- @NotBlank
- @Size
- @Min
- @Max
- @Valid

Validation should happen at the API boundary before expensive discovery work begins.

Business validation should remain in the appropriate service/domain layer.

---

## 10. Exception Handling

Use centralized exception handling.

Spring's:

@RestControllerAdvice

can provide a consistent API error format.

The backend should return controlled error responses instead of exposing stack traces or internal implementation details.

---

## 11. Asynchronous Processing

V1 discovery will use Spring's asynchronous processing capabilities.

Initial approach:

Spring Async
+
Controlled Thread Pool

This is sufficient for the first version.

The system should use:

- bounded thread pools
- timeouts
- controlled concurrency
- retries where appropriate

A message broker is not required initially.

---

## 12. HTTP Client

The backend needs an HTTP client for public-web fetching and external API communication.

The implementation can use Spring's modern HTTP client support, such as:

RestClient

or another appropriate Spring-supported client.

The HTTP layer should support:

- connection timeout
- read timeout
- redirects
- response size limits
- retry policy
- HTTP status handling

The fetching implementation should remain behind an internal abstraction so it can evolve later.

---

## 13. HTML Processing

The platform needs HTML parsing for public job pages.

A lightweight HTML parser such as:

Jsoup

can be used for:

- HTML parsing
- DOM traversal
- text extraction
- metadata extraction
- JSON-LD discovery

The parser should treat webpage content as untrusted data.

It should never execute webpage JavaScript.

---

## 14. Structured Data

The extraction layer should inspect:

- JSON-LD
- Schema.org JobPosting
- OpenGraph metadata
- HTML metadata

Structured data should be preferred where available.

However, structured data must still pass validation.

It should never automatically be considered correct simply because it exists on the page.

---

## 15. Search Provider

V1 requires at least one search/discovery provider.

The provider should be integrated behind an abstraction.

Conceptually:

SearchProvider

The rest of the backend should not depend directly on a specific provider.

This allows providers to be added or replaced later.

Provider-specific:

- API keys
- quotas
- request limits
- configuration

must remain outside business logic.

---

## 16. Configuration

Spring Boot configuration will be used.

Configuration should support:

- database connection
- search provider settings
- HTTP timeouts
- worker pool settings
- source settings
- logging
- application environment

Environment-specific configuration should be supported.

Examples:

local

test

production

---

## 17. Secrets

Secrets must not be committed to GitHub.

Examples:

- API keys
- database passwords
- provider credentials
- tokens

Use environment variables or a proper secret-management solution.

The frontend must never receive backend provider credentials.

---

## 18. Logging

Use structured application logging.

Important events include:

- discovery task creation
- source request
- fetch success/failure
- extraction result
- validation rejection
- duplicate detection
- persistence
- API errors

Logs should contain enough context to debug a discovery task.

Sensitive information should not be logged unnecessarily.

---

## 19. Monitoring

V1 should have basic application monitoring.

Spring Boot Actuator can provide:

- application health
- basic operational information
- health checks

More advanced metrics can be added once the system has real workload.

---

## 20. Testing

The backend should use automated tests from the beginning.

### Unit Tests

For:

- query generation
- URL normalization
- walk-in detection
- date extraction
- experience extraction
- normalization
- validation
- deduplication
- ranking

### Integration Tests

For:

- REST APIs
- PostgreSQL persistence
- repository behaviour
- discovery task lifecycle

### Extraction Tests

Use representative HTML fixtures to test extraction logic.

The goal is to prevent extraction changes from breaking existing source handling.

---

## 21. Test Database

Tests should not depend on a developer's local production-like database.

The test environment should use an isolated database.

A containerized PostgreSQL test environment can be introduced if useful.

The exact test setup can be finalized during implementation.

---

## 22. Frontend

The repository contains a separate:

frontend/

application.

The frontend is part of the product from Day 1, but backend development is the initial priority.

The frontend will eventually handle:

- search form
- discovery status
- result listing
- filters
- opportunity details
- source links
- loading states
- error states
- empty states

The frontend must not contain discovery logic.

---

## 23. Frontend Technology

The frontend framework can be selected based on implementation needs.

The initial architecture only requires:

- modern web application
- API integration
- responsive UI
- simple state handling

Frontend technology should not drive backend architecture.

The frontend implementation begins after the backend discovery pipeline and APIs are sufficiently stable.

---

## 24. Docker

Docker is useful for consistent local and deployment environments.

V1 can use Docker primarily for infrastructure such as:

- PostgreSQL

The backend can initially run directly during development.

A complete containerized deployment can be introduced when deployment requirements justify it.

---

## 25. Kubernetes

Kubernetes is not required for V1.

Do not introduce Kubernetes simply because the system may eventually scale.

It becomes relevant only when deployment scale and operational requirements justify it.

---

## 26. Redis

Redis is not required for the first implementation.

Possible future uses:

- caching
- rate limiting
- temporary state
- distributed coordination

Add it only when actual performance or scalability requirements justify it.

---

## 27. Kafka

Kafka is not required for V1.

The initial discovery pipeline can use:

Spring Boot
+
controlled asynchronous workers.

Kafka may become useful later when:

- discovery volume increases significantly
- multiple worker types are required
- event-driven processing becomes valuable
- independent scaling of pipeline stages is needed

It should not be introduced just for learning or architectural fashion.

---

## 28. Search Engine

A dedicated search engine is not required initially.

PostgreSQL can handle the initial opportunity search.

A search engine can be introduced later if:

- dataset size becomes large
- text search becomes more advanced
- relevance ranking becomes more complex
- search latency requires a dedicated index

PostgreSQL remains the source of truth even after a search index is introduced.

---

## 29. Microservices

V1 will use a modular monolith.

Logical modules can include:

- API
- Discovery
- Extraction
- Normalization
- Validation
- Deduplication
- Freshness
- Ranking
- Search
- Persistence

These modules should have clear boundaries inside one Spring Boot application.

They can be separated into services later if real scaling or ownership requirements justify it.

---

## 30. AI

AI is optional in V1.

The core product should work without requiring AI for every page.

Preferred extraction order:

Structured Data
→ Source Parser
→ DOM/Text Rules
→ AI Fallback

AI may later be used for:

- difficult extraction
- ambiguous walk-in detection
- semantic matching
- duplicate similarity
- classification

AI output must still pass validation.

---

## 31. Initial V1 Stack

The practical V1 stack is:

Backend:

Java 21
Spring Boot
Spring Web
Spring Validation
Spring Data JPA
Hibernate
PostgreSQL
Maven
Spring Async
RestClient
Jsoup
Spring Boot Actuator
Flyway

Frontend:

Separate web application inside:

frontend/

Infrastructure:

PostgreSQL
Docker where useful

External:

Search Provider
Public Web Sources

---

## 32. Technologies Explicitly Deferred

The following are intentionally deferred unless product requirements justify them:

- Kafka
- Kubernetes
- Redis
- Elasticsearch/OpenSearch
- Microservices
- Distributed tracing infrastructure
- Object storage for raw pages
- AI extraction for every page
- Complex event-driven architecture
- Multi-region deployment

This is a product decision, not a statement that these technologies are unnecessary forever.

---

## 33. Technology Selection Principle

Every technology should answer a real requirement.

Before adding a technology, ask:

1. What problem does it solve?
2. Does V1 actually have that problem?
3. Can the current stack solve it?
4. What operational complexity does it introduce?
5. Can it be added later without major redesign?

If the answer does not justify the additional complexity, do not add it.

---

## 34. Final V1 Architecture

The initial technical architecture is:

Frontend
    ↓
Spring Boot REST API
    ↓
Modular Backend
    ↓
Async Discovery Pipeline
    ↓
Public Search Providers / Sources
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
Ranking
    ↓
PostgreSQL

The technology stack should remain intentionally small until the product proves that additional infrastructure is necessary.

---

# Final Principle

**Build the product with the smallest reliable technology stack that can solve the real problem.**

Do not add technology because it is popular.

Add it when the product actually needs it.
