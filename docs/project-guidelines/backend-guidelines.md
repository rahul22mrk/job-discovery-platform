# Backend Project Guidelines

## 1. Purpose

This document defines the mandatory development rules for the backend of the Job Discovery Platform.

These guidelines apply to:

* Human developers
* AI coding assistants
* Code generators
* Code reviewers
* Future contributors

Any code added to the backend must follow these rules unless an explicit architecture decision overrides them.

The goal is to keep the backend:

* Clean
* Maintainable
* Testable
* Secure
* Scalable
* Understandable
* Consistent
* Production-oriented

Do not introduce a new architectural pattern, framework, library, or coding style without a clear reason.

---

# 2. Technology Baseline

The V1 backend uses:

* Java 21
* Spring Boot
* Spring Web
* Spring Validation
* Spring Data JPA
* Hibernate
* PostgreSQL
* Maven
* Flyway
* Jsoup
* Spring Async
* Spring Boot Actuator

The backend is a modular monolith.

Do not introduce microservices for V1.

Do not introduce Kafka, Redis, Kubernetes, or a dedicated search engine unless an explicit project decision requires them.

---

# 3. Core Engineering Principles

The following principles are mandatory.

## 3.1 SOLID

Code should follow SOLID principles.

### Single Responsibility Principle

A class should have one clear responsibility.

Bad:

DiscoveryService handles:

* HTTP requests
* HTML parsing
* database persistence
* ranking
* email sending

Good:

DiscoveryService coordinates discovery.

Fetcher handles fetching.

Extractor handles extraction.

Repository handles persistence.

RankingService handles ranking.

---

### Open/Closed Principle

Code should be open for extension but closed for unnecessary modification.

Example:

New source:

GreenhouseAdapter

should be added through the source abstraction instead of rewriting the entire discovery pipeline.

---

### Liskov Substitution Principle

Implementations of an abstraction must be safely usable wherever the abstraction is expected.

A source adapter should follow the contract defined by the source interface.

---

### Interface Segregation Principle

Interfaces should remain small and focused.

Do not create a huge interface containing unrelated operations.

---

### Dependency Inversion Principle

High-level business logic should depend on abstractions rather than concrete external implementations.

Example:

DiscoveryService
→ JobSource

not:

DiscoveryService
→ GreenhouseAdapter

---

# 4. Clean Code

Code should be written for readability first.

Prefer:

* meaningful names
* small methods
* focused classes
* simple control flow
* explicit responsibilities
* low coupling

Avoid:

* clever code
* unnecessary abstraction
* deeply nested conditions
* large methods
* duplicated logic
* unexplained magic values

Readable code is more important than reducing the number of lines.

---

# 5. Naming Conventions

Follow standard Java naming conventions.

## Classes

Use UpperCamelCase.

Examples:

* DiscoveryService
* OpportunityService
* JobSource
* GreenhouseAdapter

---

## Methods

Use lowerCamelCase.

Examples:

* discoverOpportunities()
* extractJob()
* normalizeLocation()
* validateOpportunity()

Method names should describe the action.

---

## Variables

Use lowerCamelCase.

Examples:

* searchContext
* opportunity
* sourceUrl
* eventDate

Avoid meaningless names such as:

* x
* temp
* obj
* data
* value

unless the context genuinely makes the meaning obvious.

---

## Constants

Use UPPER_SNAKE_CASE.

Examples:

* DEFAULT_PAGE_SIZE
* MAX_RETRY_COUNT
* DEFAULT_TIMEOUT_SECONDS

---

## Packages

Use lowercase.

Example:

com.jobdiscovery.discovery

Do not use underscores.

---

## Boolean Names

Prefer names that clearly represent a condition.

Good:

* isActive
* isVerified
* hasExpired
* supportsSearch

Avoid:

* activeFlag
* statusBoolean

---

# 6. Naming Philosophy

Names should describe business meaning.

Bad:

processData()

Good:

extractWalkInOpportunity()

Bad:

handle()

Good:

validateDiscoveryRequest()

Bad:

getInfo()

Good:

getOpportunityDetails()

Avoid abbreviations unless they are widely understood.

Examples:

Good:

url

api

id

http

Bad:

opp

src

ctx

mgr

unless the abbreviation is established consistently across the project.

---

# 7. Package Architecture

The backend should follow the agreed modular structure.

Recommended structure:

com.jobdiscovery
|
+-- api
|   +-- controller
|   +-- dto
|   +-- exception
|
+-- discovery
|   +-- service
|   +-- model
|   +-- source
|
+-- extraction
|   +-- service
|   +-- parser
|   +-- detector
|
+-- normalization
|
+-- validation
|
+-- deduplication
|
+-- freshness
|
+-- ranking
|
+-- search
|
+-- persistence
|   +-- entity
|   +-- repository
|
+-- common

The exact package structure can evolve, but responsibilities must remain separated.

---

# 8. Dependency Direction

Dependencies should generally move inward toward business logic.

Preferred:

Controller
→ Service
→ Domain / Processing
→ Repository

External systems should be isolated behind abstractions.

Example:

DiscoveryService
→ JobSource

JobSource
← SearchProviderAdapter

The business layer should not become tightly coupled to one external provider.

---

# 9. Controller Rules

Controllers must remain thin.

A controller should:

1. Receive request
2. Validate request
3. Call service
4. Return response

A controller must not:

* contain business logic
* query the database directly
* call external websites directly
* parse HTML
* perform deduplication
* calculate ranking
* contain complex transformations

Bad:

Controller
→ Repository
→ HTML parsing
→ ranking
→ response

Good:

Controller
→ Service
→ Domain processing
→ Repository
→ DTO

---

# 10. DTO Rules

Use DTOs at API boundaries.

Do not expose JPA entities directly from controllers.

Separate:

* Request DTO
* Response DTO
* Persistence Entity
* Internal Domain Model

Example:

SearchRequest

OpportunityResponse

OpportunityEntity

These have different responsibilities.

---

# 11. Service Layer

Services should contain application/business orchestration.

A service should coordinate operations rather than becoming a "god class."

If a service becomes too large, extract a focused responsibility.

Example:

DiscoveryService

may coordinate:

* SearchContext
* QueryGenerator
* JobSource
* ExtractionService
* ValidationService

It should not implement every operation itself.

---

# 12. Repository Rules

Repositories are responsible for persistence access.

Do not put business rules into repositories.

Good:

findActiveOpportunitiesByCity()

Bad:

findAndRankAndValidateEverything()

Complex business decisions belong in services/domain logic.

---

# 13. Entity Rules

JPA entities represent persistence.

They should not become the application's universal data model.

Avoid putting:

* HTTP logic
* external API logic
* extraction logic
* ranking logic

inside entities.

Entity relationships should be designed carefully to avoid unnecessary loading and circular serialization.

---

# 14. Transaction Rules

Use transactions around short database operations that require consistency.

Do not keep database transactions open while:

* calling external APIs
* fetching webpages
* waiting for retries
* performing long-running processing

Preferred:

Fetch
→ Process
→ Validate
→ Short DB transaction
→ Persist

---

# 15. Database Rules

PostgreSQL is the source of truth.

Rules:

* Use migrations for schema changes.
* Use constraints where appropriate.
* Use indexes based on actual query patterns.
* Avoid unnecessary indexes.
* Avoid N+1 queries.
* Avoid loading large datasets into memory.
* Use pagination for collection APIs.
* Use appropriate transaction boundaries.

Never manually modify production schema as the normal development process.

---

# 16. Migration Rules

All schema changes must be represented through versioned migrations.

A developer must not rely on:

"Run this SQL manually once."

The migration must be reproducible in:

* local
* test
* staging
* production

---

# 17. Null and Optional Data

Do not invent missing data.

If the source does not provide:

eventTime

then:

eventTime = null

Do not create a value based on assumptions.

Optional data should be represented explicitly.

---

# 18. External Source Rules

External sources are untrusted.

Never assume:

* webpage data is correct
* third-party API data is complete
* source content is current
* external response is safe

Every external result must pass through appropriate:

Extraction
→ Normalization
→ Validation

---

# 19. Web Fetching Rules

All public-web fetching must go through the approved fetching layer.

Business services should not directly create arbitrary HTTP requests.

Do not bypass:

* authentication
* CAPTCHA
* robots restrictions
* technical access controls

Respect source terms and access rules.

---

# 20. SSRF Protection

Any backend feature that fetches a URL must treat the URL as untrusted input.

Do not blindly fetch arbitrary user-provided URLs.

Validate:

* protocol
* allowed destination
* redirects
* private/internal network targets
* response size
* timeout

This is especially important because the product performs server-side web fetching.

OWASP identifies SSRF as a major API security risk.

---

# 21. Security

Security is not an optional final step.

Follow OWASP API security principles.

Important areas include:

* authentication when introduced
* authorization
* object-level access control
* input validation
* rate limiting
* resource consumption limits
* secure configuration
* API inventory
* safe third-party API consumption
* SSRF protection

OWASP's API Security Top 10 specifically highlights authorization, unrestricted resource consumption, SSRF, security misconfiguration, inventory management, and unsafe third-party API consumption among major API risks.

---

# 22. Configuration

Do not hardcode:

* API keys
* passwords
* tokens
* database credentials
* provider secrets
* environment-specific URLs

Use configuration and environment variables.

Secrets must never be committed to Git.

---

# 23. Logging

Logs should explain what happened without exposing sensitive information.

Log useful context such as:

* request ID
* discovery task ID
* source
* processing stage
* duration
* failure reason

Do not log:

* passwords
* API keys
* access tokens
* unnecessary personal data

---

# 24. Exception Handling

Use centralized exception handling.

Preferred:

@RestControllerAdvice

Use domain-specific exceptions where useful.

Examples:

* OpportunityNotFoundException
* DiscoveryTaskNotFoundException
* InvalidSearchRequestException
* SourceFetchException

Do not use generic `Exception` everywhere.

---

# 25. Error Responses

API errors must have a consistent structure.

Example:

{
"error": {
"code": "VALIDATION_ERROR",
"message": "Invalid search request"
}
}

Do not expose:

* stack traces
* database errors
* internal class names
* provider secrets
* implementation details

---

# 26. Validation

Validate input at the API boundary.

Use Jakarta Validation where appropriate.

Examples:

* @NotBlank
* @NotNull
* @Min
* @Max
* @Size
* @Valid

Business validation should remain separate from basic request validation.

---

# 27. Async Processing

Discovery is asynchronous.

Use:

* bounded thread pools
* controlled concurrency
* timeouts
* retry limits
* backoff

Do not create unbounded threads.

Do not create one thread per URL.

---

# 28. Concurrency

Concurrent code must be intentionally designed.

Avoid:

* shared mutable state
* unsafe static variables
* unsynchronized collections
* race-prone counters

Prefer:

* immutable objects
* local state
* thread-safe collections where required
* controlled concurrency

---

# 29. Retry Rules

Retry only when retrying makes sense.

Retry candidates:

* timeout
* temporary network failure
* temporary server failure
* temporary rate limiting

Do not endlessly retry:

* 404
* invalid URL
* permanent access denial
* unsupported content

Always use a retry limit.

---

# 30. Idempotency

Background processing may execute the same operation more than once.

Operations should therefore be designed to be idempotent where practical.

Use:

* normalized URLs
* content hashes
* unique constraints
* opportunity fingerprints
* safe update logic

Repeated processing should not create uncontrolled duplicates.

---

# 31. Deduplication

Deduplication is a business capability.

Possible signals:

* company
* title
* city
* event date
* venue
* application URL
* content similarity

Do not use only title matching.

Do not assume every similar title is the same job.

---

# 32. Freshness

Freshness is part of the domain.

Use explicit states such as:

* ACTIVE
* AGING
* EXPIRED

Do not rely only on frontend filtering.

Expired walk-in opportunities should not normally appear as active results.

---

# 33. Ranking

Ranking logic must be isolated.

Ranking can consider:

* role relevance
* technology relevance
* city
* experience
* event date
* freshness
* source quality
* completeness

Do not scatter ranking calculations across controllers, repositories, and entities.

---

# 34. Extraction Rules

Extraction should follow:

Structured Data
→ Source Parser
→ DOM
→ Text Patterns
→ AI Fallback

AI should not become the mandatory path for every page.

Never allow AI output to bypass validation.

---

# 35. AI Coding Assistant Rules

Any AI tool working on this repository must follow this document.

AI must:

1. Read the relevant project documentation before changing architecture.
2. Follow existing package structure.
3. Reuse existing abstractions.
4. Avoid creating duplicate utilities.
5. Avoid introducing unnecessary dependencies.
6. Avoid changing architecture without approval.
7. Preserve existing API contracts unless explicitly asked to change them.
8. Add or update tests for changed behaviour.
9. Explain significant architectural changes.
10. Prefer the simplest solution that satisfies the requirement.

AI must not:

* invent new architecture
* introduce microservices without approval
* add Kafka just because asynchronous processing exists
* add Redis without a demonstrated need
* create unnecessary design patterns
* duplicate existing services
* expose entities directly from APIs
* hardcode secrets
* bypass security controls
* silently change API contracts

---

# 36. Design Pattern Rules

Design patterns should solve real problems.

Allowed when justified:

* Strategy
* Factory
* Adapter
* Builder
* Observer
* Chain of Responsibility
* Template Method
* Specification

Do not use patterns only to make code "look enterprise."

Example:

Multiple source implementations justify:

Strategy / Adapter

A single implementation does not automatically justify creating five abstraction layers.

---

# 37. Factory Rules

Use a Factory when object creation has meaningful variation.

Do not create:

JobFactory

if it only does:

return new Job();

The abstraction must solve an actual problem.

---

# 38. Strategy Rules

Strategy is appropriate when behaviour varies by type.

Example:

JobSource

with implementations:

SearchEngineSource

GreenhouseSource

LeverSource

This is a good use of Strategy/Adapter-style design.

---

# 39. Builder Rules

Use Builder when an object has:

* many optional fields
* complex construction
* readability benefits

Do not automatically use Builder for every POJO.

---

# 40. Singleton Rules

Do not manually implement Singleton unless there is a specific need.

Spring-managed beans are already managed by the container.

Avoid custom Singleton implementations.

---

# 41. Utility Classes

Do not create a large:

Utils

class.

Prefer focused utilities.

Bad:

CommonUtils

Good:

UrlNormalizer

DateParser

ExperienceParser

Each should have a clear purpose.

---

# 42. Comments

Comments should explain:

* why something is done
* important business rules
* non-obvious constraints
* external-provider quirks

Do not comment obvious code.

Bad:

// increment counter
counter++;

Good:

// Retry only transient provider failures because permanent failures
// should not consume the source quota.

---

# 43. Documentation

Public or important classes should have useful documentation where necessary.

Document:

* non-obvious contracts
* important business rules
* external integration behaviour
* architectural decisions

Do not write documentation that merely repeats the method name.

---

# 44. Testing Standard

Every meaningful business behaviour should have tests.

At minimum, changes should consider:

* happy path
* invalid input
* edge cases
* failure path
* duplicate case
* null/missing data where relevant

Do not test implementation details unnecessarily.

Test observable behaviour.

---

# 45. Unit Tests

Unit tests should be:

* focused
* deterministic
* fast
* isolated

A unit test should ideally test one meaningful behaviour.

Avoid large tests that depend on many unrelated classes.

---

# 46. Integration Tests

Use integration tests for:

* database behaviour
* API contracts
* persistence
* transaction behaviour
* important external integration boundaries

Do not replace all unit tests with integration tests.

---

# 47. Code Formatting

Use a consistent formatter.

The project should enforce formatting automatically where practical.

Do not manually debate formatting in code review.

Formatting should be handled by tooling.

---

# 48. Static Analysis

The project should use automated quality checks where practical.

Potential checks:

* compilation
* tests
* formatting
* static analysis
* dependency vulnerabilities

A developer should not rely only on manual review to find obvious problems.

---

# 49. Dependency Rules

Before adding a dependency, ask:

1. Do we actually need it?
2. Can Spring Boot/JDK already solve the problem?
3. Is the dependency actively maintained?
4. Does it introduce security or operational risk?
5. Is the dependency worth its complexity?

Do not add libraries for trivial functionality.

---

# 50. API Compatibility

Do not silently break existing APIs.

Before changing an API:

* check consumers
* update DTOs
* update tests
* update documentation
* consider versioning

V1 API contracts should remain stable once frontend integration begins.

---

# 51. Git and Commit Rules

Commits should be:

* small
* focused
* understandable

Good:

feat: add discovery task API

fix: handle expired walk-in opportunities

refactor: isolate source adapter

Bad:

changes

update

final

Use conventional commit-style prefixes where practical:

* feat
* fix
* refactor
* test
* docs
* chore
* perf
* security

---

# 52. Pull Request Rules

A meaningful change should be reviewable as one logical unit.

A PR should explain:

* What changed
* Why it changed
* How it was tested
* Any architectural impact
* Any migration required

Avoid mixing unrelated changes.

Google's engineering guidance emphasizes design, functionality, complexity, testing, naming, documentation, and overall code health during review.

---

# 53. AI Change Workflow

When an AI assistant is asked to implement a feature, it must follow:

Understand
→ Inspect existing code
→ Identify affected module
→ Propose approach
→ Implement
→ Test
→ Review
→ Report changes

The AI should not immediately generate a large amount of code without understanding the existing structure.

---

# 54. Before Creating a New Class

Ask:

* Does this responsibility already exist?
* Can an existing service be extended?
* Is a new abstraction really necessary?
* Does the class have one clear responsibility?
* Will this reduce or increase complexity?

Do not create classes only to satisfy a pattern.

---

# 55. Before Creating a New Package

Ask:

* Is there a real responsibility boundary?
* Are multiple classes likely to belong here?
* Does this package improve discoverability?
* Is the complexity justified?

Avoid package explosion.

---

# 56. Before Adding a New Technology

Ask:

* What real problem are we solving?
* Can the existing stack solve it?
* Is this required for V1?
* What operational complexity does it add?
* Can it be introduced later?

Default answer should be:

Keep the stack simple.

---

# 57. Production Readiness

Before considering a feature complete, verify:

* validation
* error handling
* logging
* tests
* security
* database behaviour
* concurrency
* failure handling
* observability
* API compatibility

A feature is not complete just because the happy path works.

---

# 58. Definition of Done

A backend feature is considered done when:

* implementation follows the architecture
* naming is consistent
* SOLID principles are respected
* unnecessary complexity is avoided
* tests exist
* validation exists where required
* error handling exists
* security implications are considered
* logs are useful
* database changes use migrations
* API contracts are documented
* code builds successfully
* tests pass

---

# 59. Golden Rules

These rules have highest priority:

1. Do not invent architecture.
2. Do not introduce unnecessary technologies.
3. Keep responsibilities separate.
4. Prefer simple code over clever code.
5. Follow SOLID where it provides real value.
6. Use design patterns only when justified.
7. Never expose database entities directly through APIs.
8. Never trust external data blindly.
9. Never invent missing job information.
10. Never bypass security or access controls.
11. Keep external integrations behind abstractions.
12. Make background processing safe and idempotent.
13. Test meaningful behaviour.
14. Keep API contracts stable.
15. Improve code health with every meaningful change.

---

# Final Backend Rule

When there is a choice between:

A complicated "enterprise-looking" solution

and

A simple, maintainable solution that satisfies the actual requirement,

choose the simple solution.

The backend should evolve because the product needs it, not because a technology or design pattern looks impressive.
