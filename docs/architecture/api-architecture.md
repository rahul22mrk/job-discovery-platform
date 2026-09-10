# API Architecture

## Purpose

The API is the boundary between the frontend and backend of the Job Discovery Platform.

The frontend should communicate with the backend only through well-defined APIs.

The API layer should not contain discovery, extraction, normalization, deduplication, or ranking logic.

Its responsibility is to:

- Receive requests
- Validate input
- Call the appropriate backend service
- Return consistent responses
- Handle errors
- Expose discovery status
- Expose job opportunities
- Support pagination and filtering

V1 uses REST APIs.

---

## 1. API Base Path

All V1 APIs should use:

/api/v1

Versioning from the beginning makes future API changes easier.

Example:

/api/v1/discovery/search

---

## 2. API Architecture

The request flow is:

Frontend
→ REST Controller
→ Service Layer
→ Domain / Discovery Logic
→ Repository
→ PostgreSQL

For discovery:

Frontend
→ Discovery API
→ Discovery Service
→ Discovery Task
→ Background Pipeline
→ PostgreSQL

The frontend should never communicate directly with:

- PostgreSQL
- Search providers
- Job websites
- Crawlers
- Extraction services

---

## 3. V1 API Groups

V1 initially needs four main API areas:

- Discovery
- Opportunities
- Health
- Future administrative/internal APIs

The public product APIs should remain small.

Do not expose internal pipeline operations unnecessarily.

---

# Discovery APIs

## 4. Start Discovery

Endpoint:

POST /api/v1/discovery/search

Purpose:

Start a new discovery operation based on the user's search criteria.

Example request:

{
  "city": "Bengaluru",
  "role": "Java Backend Developer",
  "skills": [
    "Java",
    "Spring Boot"
  ],
  "minExperience": 2,
  "maxExperience": 5,
  "fromDate": "2026-09-11",
  "toDate": "2026-10-11"
}

The request should contain only information required to define the search.

The backend should create a discovery task.

The API should not wait for the complete internet discovery process.

---

## 5. Discovery Response

A successful discovery-start request should return a task/search identifier.

Example:

{
  "searchId": "generated-id",
  "status": "CREATED"
}

The exact identifier format can be decided during implementation.

The frontend uses this identifier to check discovery progress.

---

## 6. Discovery Status

Endpoint:

GET /api/v1/discovery/search/{searchId}

Purpose:

Return the current state of a discovery task.

Possible statuses:

- CREATED
- RUNNING
- COMPLETED
- PARTIALLY_COMPLETED
- FAILED

Example response:

{
  "searchId": "generated-id",
  "status": "RUNNING",
  "progress": {
    "urlsDiscovered": 45,
    "urlsFetched": 31,
    "opportunitiesFound": 8
  }
}

Progress information is optional in early V1.

The most important information is the task status.

---

## 7. Discovery Completion

When discovery completes, the task should expose enough information for the frontend to retrieve results.

Example:

{
  "searchId": "generated-id",
  "status": "COMPLETED",
  "resultCount": 12
}

The frontend can then call the opportunity API.

The discovery status API should not return the complete result dataset.

This keeps responsibilities separate.

---

# Opportunity APIs

## 8. List Opportunities

Endpoint:

GET /api/v1/opportunities

Purpose:

Return discovered job opportunities.

Possible query parameters:

- city
- role
- skill
- minExperience
- maxExperience
- fromDate
- toDate
- type
- freshness
- page
- size
- sort

Example:

GET /api/v1/opportunities?city=Bengaluru&skill=Java&page=0&size=20

The API should return only opportunities that satisfy the requested filters.

---

## 9. Opportunity Response

Example:

{
  "content": [
    {
      "id": "generated-id",
      "company": {
        "id": "company-id",
        "name": "Example Technologies"
      },
      "title": "Java Backend Developer",
      "city": "Bengaluru",
      "experience": {
        "min": 2,
        "max": 5
      },
      "event": {
        "date": "2026-09-15",
        "startTime": "10:00",
        "endTime": "16:00",
        "venue": "Bengaluru"
      },
      "skills": [
        "Java",
        "Spring Boot"
      ],
      "type": "WALK_IN",
      "freshness": "ACTIVE",
      "verificationStatus": "SOURCE_CONFIRMED"
    }
  ],
  "page": 0,
  "size": 20,
  "totalElements": 1,
  "totalPages": 1
}

The actual response structure can be refined during implementation.

---

## 10. Opportunity Details

Endpoint:

GET /api/v1/opportunities/{id}

Purpose:

Return complete information about one opportunity.

Possible response fields:

- id
- company
- title
- description
- city
- country
- job type
- experience
- skills
- event date
- event time
- venue
- application information
- freshness
- verification status
- source information
- discovered timestamp
- last seen timestamp

The details API should provide enough information for the user to decide whether to apply or attend.

---

## 11. Source Information

Source transparency is a core product requirement.

Opportunity details should expose source information.

Example:

{
  "sources": [
    {
      "name": "Company Career Page",
      "type": "COMPANY_CAREER_PAGE",
      "url": "source-url",
      "isPrimary": true
    }
  ]
}

The frontend should provide a clear way to open the original source.

The platform should not hide the original source behind unnecessary redirects.

---

# Search and Filtering

## 12. City Filter

Example:

GET /api/v1/opportunities?city=Bengaluru

The backend should use normalized city values internally.

The API may accept common variations where useful.

Example:

Bangalore

can be normalized internally to:

Bengaluru

---

## 13. Skill Filter

Example:

GET /api/v1/opportunities?skill=Java

Skill matching should use normalized skills.

Future versions can support:

- multiple skills
- AND matching
- OR matching
- semantic skill matching

V1 should keep the behaviour simple and predictable.

---

## 14. Experience Filter

Example:

GET /api/v1/opportunities?minExperience=2&maxExperience=5

The API should return opportunities compatible with the requested experience range.

Experience matching should be handled by the backend domain/search layer.

---

## 15. Date Filter

Example:

GET /api/v1/opportunities?fromDate=2026-09-11&toDate=2026-10-11

Date filtering should operate on the walk-in/event date.

Expired opportunities should not appear in active searches unless explicitly requested.

---

## 16. Freshness Filter

Example:

GET /api/v1/opportunities?freshness=ACTIVE

Possible values:

- ACTIVE
- AGING
- EXPIRED

The default V1 search should prefer active opportunities.

---

## 17. Pagination

Opportunity APIs should always support pagination.

Example:

page=0
size=20

The backend should enforce a maximum page size.

For example:

Requested size:
1000

Backend maximum:
100

The backend returns at most 100 records.

The exact maximum can be configured.

This protects the API from unnecessarily large responses.

---

## 18. Sorting

Possible V1 sorting options:

- relevance
- eventDate
- freshness
- newest

Example:

sort=relevance

Relevance should be the default for discovery results.

The ranking layer determines relevance.

The controller should not implement ranking logic itself.

---

# API Validation

## 19. Request Validation

The backend should validate incoming requests.

Examples:

Invalid city:
reject request

Negative experience:
reject request

Minimum experience greater than maximum:
reject request

Invalid date range:
reject request

Empty discovery request:
reject request

Validation should happen before starting expensive discovery work.

---

## 20. Validation Error

All validation errors should use a consistent response structure.

Example:

{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid search request",
    "details": [
      {
        "field": "minExperience",
        "message": "Minimum experience cannot be negative"
      }
    ]
  }
}

The exact error codes can be finalized during backend implementation.

---

# Error Handling

## 21. Error Response Structure

The API should use a consistent error format.

Example:

{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "Opportunity not found"
  }
}

Possible error codes:

- VALIDATION_ERROR
- RESOURCE_NOT_FOUND
- INVALID_REQUEST
- DISCOVERY_FAILED
- INTERNAL_ERROR
- SERVICE_UNAVAILABLE

Internal exception details should not be exposed to the client.

---

## 22. HTTP Status Codes

Use standard HTTP status codes.

Examples:

200 OK
→ successful GET

201 Created
→ resource/task created

202 Accepted
→ asynchronous discovery accepted

400 Bad Request
→ invalid request

404 Not Found
→ resource does not exist

409 Conflict
→ request conflicts with current state

429 Too Many Requests
→ rate limit exceeded

500 Internal Server Error
→ unexpected backend error

503 Service Unavailable
→ temporary service failure

The exact status mapping can be finalized during implementation.

---

# Asynchronous Discovery

## 23. Why Discovery Is Asynchronous

Web discovery can involve:

- multiple search queries
- many URLs
- HTTP requests
- redirects
- parsing
- extraction
- validation
- deduplication

The API should not keep the initial HTTP request open for the entire process.

Instead:

POST /discovery/search
→ create task
→ return searchId

Then:

GET /discovery/search/{searchId}
→ check status

Finally:

GET /opportunities
→ retrieve results

This keeps the API responsive.

---

## 24. Discovery Failure

If some external sources fail, the discovery task should not automatically fail.

For example:

10 sources attempted
2 failed
8 succeeded
15 opportunities discovered

The task can be:

PARTIALLY_COMPLETED

This provides better product behaviour than returning zero results because one source failed.

---

# Response Design

## 25. DTOs

The API should use DTOs.

Examples:

SearchRequest
SearchResponse
DiscoveryStatusResponse
OpportunityResponse
OpportunitySummaryResponse
SourceResponse
CompanyResponse
ErrorResponse
PageResponse

JPA entities should not be returned directly from controllers.

---

## 26. Opportunity Summary vs Details

The list API should return a lightweight summary.

The details API can return more information.

List response:

- title
- company
- city
- experience
- event date
- event time
- freshness
- key skills
- primary source

Details response:

- all summary fields
- full description
- venue
- application information
- all sources
- timestamps
- additional metadata

This keeps list responses smaller.

---

## 27. Source URL Handling

Source URLs should be returned as actual source URLs.

The frontend should open them directly or through a controlled redirect only if there is a product/security reason.

The platform should not replace the source URL with an internal page that hides the original source.

---

# Security

## 28. Input Safety

The API must treat all user input as untrusted.

Validate:

- strings
- dates
- numbers
- lists
- URLs where accepted

Avoid directly constructing database queries from user input.

Use parameterized queries and framework-supported query mechanisms.

---

## 29. Rate Limiting

V1 may not require sophisticated distributed rate limiting.

However, the API should have basic protection against excessive requests.

Potential future implementation:

Redis-based rate limiting

For V1, a simple application-level mechanism can be sufficient if required.

---

## 30. CORS

The backend should allow requests only from configured frontend origins.

Do not use unrestricted production CORS unless there is a clear reason.

Development and production CORS configuration should be separate.

---

# Observability

## 31. Request Logging

Important API information should be logged:

- endpoint
- request identifier
- status
- duration
- error code

Sensitive user information should not be unnecessarily logged.

---

## 32. Request ID

The backend should support a request/correlation identifier.

Example:

X-Request-Id

This helps trace a request across:

Frontend
→ API
→ Discovery Task
→ Pipeline
→ Database

The exact header strategy can be finalized during implementation.

---

# API Versioning

## 33. V1

All initial APIs use:

/api/v1

Example:

/api/v1/discovery/search

When breaking changes are required later, a new version can be introduced.

Example:

/api/v2/...

The initial V1 API should avoid unnecessary complexity.

---

# Health API

## 34. Health Check

A simple health endpoint should exist for deployment and monitoring.

Example:

GET /actuator/health

Spring Boot Actuator can provide the implementation.

The health endpoint should allow infrastructure to determine whether the application is running correctly.

---

# Internal vs Public APIs

## 35. Public Product APIs

V1 public APIs:

POST /api/v1/discovery/search

GET /api/v1/discovery/search/{searchId}

GET /api/v1/opportunities

GET /api/v1/opportunities/{id}

These are enough to support the initial frontend.

---

## 36. Internal APIs

The backend should avoid exposing internal pipeline operations as HTTP APIs unless required.

For example, there is no initial need for public endpoints such as:

POST /crawl
POST /extract
POST /normalize
POST /deduplicate

These are internal backend responsibilities.

---

# API and Backend Boundary

## 37. Controller Responsibility

Controllers should:

- receive requests
- validate DTOs
- call services
- return responses

Controllers should not:

- fetch webpages
- parse HTML
- detect walk-ins
- normalize jobs
- perform deduplication
- calculate ranking
- directly access external search providers

---

## 38. Service Responsibility

Services should contain application/business logic.

Example:

DiscoveryService

can coordinate:

- task creation
- query generation
- discovery execution
- result processing

OpportunityService

can handle:

- opportunity retrieval
- filtering
- pagination

RankingService

handles ranking.

ExtractionService

handles extraction.

The API layer should remain thin.

---

# Frontend Integration

## 39. Frontend Flow

The initial frontend flow is:

1. User enters search criteria
2. Frontend calls POST /api/v1/discovery/search
3. Backend returns searchId
4. Frontend shows discovery/loading state
5. Frontend checks discovery status
6. Discovery completes
7. Frontend requests opportunities
8. Frontend displays results
9. User opens opportunity details
10. User opens original source

The frontend should not know how discovery works internally.

---

## 40. Empty Results

An empty result should be treated as a valid response.

Example:

{
  "content": [],
  "page": 0,
  "size": 20,
  "totalElements": 0,
  "totalPages": 0
}

The frontend should be able to show a useful message such as:

"No relevant walk-in opportunities found for this search."

The backend should not manufacture results simply to avoid an empty state.

---

# Future API Expansion

## 41. Future Regular Jobs

Future APIs may support:

GET /api/v1/jobs

The existing opportunity model should make this possible without breaking V1.

---

## 42. Future Company Search

Future example:

GET /api/v1/companies/{companyId}/opportunities

This can support requests such as:

"Show me opportunities from Amazon."

---

## 43. Future Saved Searches

Future APIs may include:

POST /api/v1/saved-searches

GET /api/v1/saved-searches

DELETE /api/v1/saved-searches/{id}

These are intentionally outside V1.

---

## 44. Future Notifications

Future APIs may include:

GET /api/v1/notifications

Notification delivery itself should remain a separate backend concern.

The discovery pipeline should not become tightly coupled to notification delivery.

---

# API Evolution Principles

## 45. Stable Contracts

Once frontend integration begins, API contracts should change carefully.

Breaking changes should require:

- versioning
- migration
- compatibility planning

---

## 46. Do Not Expose Internal Models

Database entities and internal pipeline models should not automatically become API models.

The API should expose only what the product needs.

---

## 47. Keep V1 Small

V1 does not need dozens of endpoints.

The goal is to support:

Search
→ Discover
→ View Results
→ View Details
→ Open Source

Everything else can come later.

---

# Final V1 API Surface

The initial API surface is:

### Discovery

POST /api/v1/discovery/search

GET /api/v1/discovery/search/{searchId}

### Opportunities

GET /api/v1/opportunities

GET /api/v1/opportunities/{id}

### Health

GET /actuator/health

This small API surface is enough to support the first complete product flow.

---

# Final Architecture

The final API flow is:

Frontend
|
v
REST API
|
+-- Discovery Controller
|      |
|      v
|   Discovery Service
|      |
|      v
|   Discovery Task
|      |
|      v
|   Background Pipeline
|
+-- Opportunity Controller
       |
       v
    Opportunity Service
       |
       v
    PostgreSQL

The API layer remains a clean boundary between the frontend and backend.

The backend owns discovery and data processing.

The frontend consumes stable APIs.

The core V1 product flow remains:

Search
→ Discover
→ Process
→ Store
→ Retrieve
→ View
→ Open Original Source
