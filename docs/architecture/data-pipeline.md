# Data Pipeline

## 1. Purpose

The data pipeline is the core of the Job Discovery Platform.

Its job is to take a user's search intent and turn it into clean, relevant, fresh job opportunities.

The basic flow is:

    User Search
        ↓
    Search Context
        ↓
    Query Generation
        ↓
    Source Discovery
        ↓
    URL Normalization
        ↓
    Page Fetching
        ↓
    Raw Document
        ↓
    Text / Structured Data Extraction
        ↓
    Walk-in Detection
        ↓
    Job Extraction
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
    Database
        ↓
    Search Results

The pipeline should be modular so that individual stages can be improved without rewriting the complete system.

---

# 2. Pipeline Principles

The pipeline should follow these principles:

- Do not trust discovered data blindly.
- Preserve the original source.
- Do not invent missing information.
- Separate raw data from normalized data.
- Make every stage observable.
- One failed page should not stop the complete pipeline.
- Deduplicate before showing results.
- Treat freshness as a first-class concern.
- Keep source-specific logic isolated.
- Prefer deterministic processing where it is sufficient.
- Use AI only where it provides measurable value.

---

# 3. Input

The pipeline starts with a user's search request.

Example:

    {
      "city": "Bengaluru",
      "keyword": "Java",
      "experienceMin": 3,
      "experienceMax": 5,
      "fromDate": "2026-09-01",
      "toDate": "2026-09-30",
      "type": "WALK_IN"
    }

The API converts this request into an internal SearchContext.

---

# 4. Search Context

SearchContext is the normalized representation of the user's intent.

It may contain:

- City
- Country
- Company
- Keyword
- Skills
- Opportunity type
- Minimum experience
- Maximum experience
- Start date
- End date

The SearchContext should be independent of any particular search provider.

---

# 5. Query Generation

The Query Generator converts SearchContext into discovery queries.

Example:

    Input:

    Bengaluru + Java + WALK_IN

    Queries:

    "walk-in interview" "Bengaluru" "Java"
    "walk-in drive" "Bengaluru" "Java"
    "walkin interview" "Bengaluru" "Java"
    "walk-in recruitment" "Bengaluru" "Java"
    "Java developer" "walk-in" "Bengaluru"
    "Java backend" "walk-in" "Bengaluru"

The generator should support:

- Technology aliases
- Role aliases
- City aliases
- Walk-in terminology
- Date-related terms when useful

It should also remove duplicate queries.

---

# 6. Source Discovery

The discovery layer searches public sources for candidate pages.

Possible sources include:

- Search providers
- Company career pages
- Public ATS
- Job boards
- Aggregators
- Public APIs
- RSS/XML feeds
- Sitemaps

The system should use a source abstraction.

Example:

    JobSource

Possible implementations:

    SearchEngineSource
    CompanyCareerSource
    JobBoardSource
    ApiJobSource
    RssSource

The pipeline should not depend on one source permanently.

---

# 7. Candidate URLs

The discovery stage returns candidate URLs.

Example:

    https://example.com/job/java-walkin-bengaluru
    https://example.com/careers/java-developer
    https://example.com/walkin-drive

Each candidate should contain basic metadata such as:

- URL
- Source
- Discovery query
- Discovery timestamp

---

# 8. URL Normalization

Before fetching pages, URLs should be normalized.

Possible operations:

- Normalize scheme
- Normalize hostname
- Remove safe tracking parameters
- Normalize trailing slash where appropriate
- Detect exact duplicates

The system must be careful not to remove parameters that are required to identify the actual page.

---

# 9. Page Fetching

The Fetcher retrieves publicly accessible pages.

Input:

    URL

Output:

    RawDocument

The fetcher should handle:

- HTTP status
- Redirects
- Timeout
- Connection failure
- Response size
- Content type
- Retry
- Backoff

A single fetch failure should not terminate the discovery task.

---

# 10. Access Rules

The system should process only pages that are appropriately accessible.

It should respect applicable:

- robots.txt rules
- Terms of service
- Access restrictions
- Rate limits

The system must not bypass:

- CAPTCHA
- Login requirements
- Paywalls
- Authentication
- Technical access controls

If a source cannot be accessed appropriately, the pipeline should record the failure and continue.

---

# 11. Raw Document

The fetched page should be represented separately from the final job posting.

Example:

    RawDocument

    sourceUrl
    sourceName
    rawContent
    contentType
    httpStatus
    discoveredAt
    fetchedAt
    contentHash

Raw data is useful for:

- Debugging
- Reprocessing
- Improving extraction
- Comparing page changes
- Investigating incorrect results

---

# 12. Content Cleaning

Raw HTML is converted into useful text.

Pipeline:

    Raw HTML
       ↓
    Remove scripts
       ↓
    Remove styles
       ↓
    Remove irrelevant elements
       ↓
    Extract visible content
       ↓
    Normalize whitespace
       ↓
    Clean Text

The cleaner should preserve information that is useful for job extraction.

---

# 13. Structured Data Extraction

Before relying only on page text, the pipeline should inspect structured data.

Important signals include:

- JSON-LD
- JobPosting structured data
- OpenGraph metadata
- Other structured metadata

JobPosting structured data may contain information such as:

- Job title
- Hiring organization
- Location
- Employment type
- Date posted
- Description
- Application information

Structured data should be treated as an extraction signal, not automatic proof that the information is correct or current.

---

# 14. Walk-in Detection

The pipeline next determines whether the page represents a walk-in opportunity.

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

Classification:

    WALK_IN
    NOT_WALK_IN
    UNCERTAIN

The detector should combine multiple signals.

The presence of the word "walk-in" alone should not be sufficient.

---

# 15. Job Extraction

If the page is a strong walk-in candidate, extract structured information.

Target fields:

- Company
- Job title
- Description
- City
- Country
- Opportunity type
- Event date
- Event start time
- Event end time
- Venue
- Experience
- Skills
- Application information
- Source URL
- Source name

The extractor should support multiple strategies.

Priority:

    Structured Data
         ↓
    Source-specific Parser
         ↓
    DOM Extraction
         ↓
    Text Patterns
         ↓
    AI-assisted Extraction

Not every page requires AI.

---

# 16. Raw vs Normalized Data

Raw data and normalized data should remain separate.

Example:

    Raw Page
       ↓
    Raw Job Data
       ↓
    Normalized Job Posting

Raw values may contain:

    "Bangalore"

while normalized data may contain:

    "Bengaluru"

The original information should still be preserved where useful.

---

# 17. Normalization

Normalization converts different representations into a common format.

Examples:

    Bangalore → Bengaluru

    3-5 years → min=3, max=5

    3+ years → min=3

Technology normalization:

    Core Java
    Java Developer
    Java Backend
    Java Engineer

can map to a common technology representation for matching.

Normalization should be deterministic wherever possible.

---

# 18. Validation

Validation checks whether extracted information is usable.

Minimum checks may include:

- Company exists
- Job title exists
- Source URL exists
- Walk-in evidence exists
- Location is valid
- Event date is valid when present
- Experience is not corrupted
- Opportunity is not already expired

Invalid data should be rejected or marked appropriately.

The system should never fill missing values by guessing.

---

# 19. Deduplication

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
    Aggregator A

These should normally become one logical opportunity.

Initial duplicate signals:

- Company
- Normalized job title
- City
- Event date
- Venue
- Application URL

More advanced similarity can be introduced later.

---

# 20. Source Linking

One opportunity can have multiple sources.

Example:

    Opportunity
       ├── Company Website
       ├── Job Board
       └── Aggregator

The system should retain these relationships.

This provides:

- Source transparency
- Better trust signals
- Better duplicate detection
- Better source quality measurement

---

# 21. Freshness

Walk-in information changes quickly.

The pipeline should track:

- firstSeenAt
- discoveredAt
- lastCheckedAt
- eventDate
- status

Possible status:

    ACTIVE
    AGING
    EXPIRED

Basic rule:

    If eventDate is before the current date,
    mark the opportunity as EXPIRED.

Future states may include:

    CANCELLED
    RESCHEDULED

---

# 22. Change Detection

Change detection is not required for the first implementation, but the pipeline should leave room for it.

Example:

    Old:
    Interview Date = 15 September

    New:
    Interview Date = 20 September

The platform could later detect the change and update the opportunity.

This becomes important when the same opportunity remains online but its details change.

---

# 23. Trust Signals

The pipeline can generate internal trust signals.

Examples:

- Company source found
- Multiple independent sources
- Event date confirmed
- Venue confirmed
- Application link available
- Recently checked
- Source has good historical quality

Important distinction:

    Extraction Confidence
    =
    How confident the system is that extraction is correct.

    Verification
    =
    Evidence that the opportunity/details have been independently confirmed.

These should not be treated as the same thing.

---

# 24. Ranking

After validation and deduplication, opportunities can be ranked.

Initial ranking signals:

- Technology match
- Role match
- City match
- Experience match
- Event date relevance
- Freshness
- Source quality
- Completeness
- Trust signals

The first version should use a simple transparent scoring system.

---

# 25. Persistence

The final normalized opportunity is stored in PostgreSQL.

Important entities:

    Company
    JobPosting
    JobSource
    RawDocument
    DiscoveryTask
    Skill

The database becomes the source for the user-facing search layer.

---

# 26. Complete Pipeline

The complete V1 pipeline is:

    Search Request
        ↓
    SearchContext
        ↓
    Query Generator
        ↓
    Source Discovery
        ↓
    Candidate URLs
        ↓
    URL Normalization
        ↓
    Page Fetching
        ↓
    Raw Document
        ↓
    Content Cleaning
        ↓
    Structured Data Extraction
        ↓
    Walk-in Detection
        ↓
    Job Extraction
        ↓
    Normalization
        ↓
    Validation
        ↓
    Deduplication
        ↓
    Source Linking
        ↓
    Freshness
        ↓
    Ranking
        ↓
    PostgreSQL
        ↓
    Search API
        ↓
    User

---

# 27. Failure Handling

Every pipeline stage should handle failures independently.

Example:

    URL 1 → Success
    URL 2 → Fetch Failed
    URL 3 → Success
    URL 4 → Extraction Failed

The task should still complete.

The system should record:

- Stage
- URL
- Error
- Timestamp
- Retry information

Failures should be observable rather than silently ignored.

---

# 28. Idempotency

Running the same discovery task multiple times should not create duplicate opportunities.

The pipeline should use:

- URL normalization
- Content hashes
- Opportunity identity
- Database constraints
- Duplicate detection

Idempotency becomes especially important when asynchronous processing and retries are introduced.

---

# 29. Observability

Track pipeline-level metrics.

Examples:

    URLs discovered
    URLs fetched
    Fetch failures
    Pages parsed
    Walk-in candidates
    Valid opportunities
    Invalid opportunities
    Duplicates
    Expired opportunities
    Average processing time

These metrics will help identify where the system is losing quality.

---

# 30. Future Pipeline Evolution

V1:

    Search Provider
        ↓
    Fetch
        ↓
    Extract
        ↓
    Validate
        ↓
    Deduplicate
        ↓
    Store

Future:

    Multiple Discovery Sources
        ↓
    Source Prioritization
        ↓
    Distributed Fetching
        ↓
    Extraction Workers
        ↓
    Change Detection
        ↓
    Verification
        ↓
    Search Index
        ↓
    Personalization

The architecture should evolve without changing the core domain model.

---

# 31. Key Principle

The pipeline should optimize for:

    Relevance
        >
    Freshness
        >
    Trust
        >
    Quantity

The goal is not to collect the maximum number of pages.

The goal is to produce the maximum number of useful opportunities from the collected data.
