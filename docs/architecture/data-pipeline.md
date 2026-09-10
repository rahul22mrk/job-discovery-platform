# Data Pipeline

## Purpose

The data pipeline is the core of the Job Discovery Platform.

Its job is to take a user's search request and automatically discover, process, validate, and rank relevant job opportunities from the public internet.

The pipeline must prioritize:

- Relevance over quantity
- Freshness over volume
- Trust over hype
- Source transparency
- Correctness over aggressive extraction
- Failure isolation
- Scalability without unnecessary complexity

The backend owns the complete discovery pipeline.

The frontend only sends search requests and consumes the processed results through APIs.

---

## High-Level Flow

The V1 pipeline is:

Search Request
→ Search Context
→ Query Generation
→ Source Discovery
→ URL Normalization
→ Page Fetching
→ Raw Document
→ Text / Structured Data Extraction
→ Walk-in Detection
→ Job Extraction
→ Normalization
→ Validation
→ Deduplication
→ Freshness
→ Ranking
→ Persistence
→ Search API
→ Frontend

Each stage should have a clear responsibility.

A failure in one source or one page should not stop the complete discovery task.

---

## 1. Search Request

The user starts a discovery search from the frontend.

Example:

- City: Bengaluru
- Technology: Java
- Role: Backend Developer
- Experience: 2-5 years
- Date range: Next 30 days

The frontend sends the request to the backend.

Example endpoint:

POST /api/v1/discovery/search

The backend should validate the request before starting discovery.

---

## 2. Search Context

The backend converts the request into an internal `SearchContext`.

Possible fields:

- city
- country
- company
- keyword
- role
- technologies
- skills
- job type
- minimum experience
- maximum experience
- date range
- source preferences
- search options

Example:

City:
Bengaluru

Role:
Java Backend Developer

Skills:
Java, Spring Boot, Microservices

Experience:
2-5 years

Date range:
Next 30 days

The SearchContext is used by later stages of the pipeline.

---

## 3. Query Generation

The backend generates multiple search queries instead of depending on one query.

For example:

"walk-in interview" "Bengaluru" "Java"

"walk-in drive" "Bengaluru" "Java"

"walkin interview" "Bengaluru" "Java"

"walk-in recruitment" "Bengaluru" "Java"

"Java developer" "walk-in" "Bengaluru"

"Java backend" "walk-in" "Bengaluru"

Queries can later become more intelligent based on:

- role
- technology
- location
- experience
- company
- source
- historical result quality

Query generation should remain independent from the search provider.

---

## 4. Source Discovery

The discovery engine uses configured search providers and public sources to find potentially relevant URLs.

Possible sources include:

- Search engines
- Search APIs
- Company career pages
- Public ATS pages
- Job boards
- Job aggregators
- Public feeds
- Sitemaps
- Public APIs

The system should not assume that one source contains all relevant jobs.

The architecture must therefore support multiple source adapters.

---

## 5. Source Abstraction

External sources should be isolated behind a common interface.

Conceptually:

JobSource

Responsibilities may include:

- accepting a discovery query
- searching the source
- returning candidate URLs
- identifying the source
- handling source-specific behaviour

The discovery pipeline should not contain source-specific logic everywhere.

For example:

SearchProviderAAdapter
SearchProviderBAdapter
GreenhouseAdapter
LeverAdapter

can implement the same source abstraction.

This makes it possible to add or replace sources without redesigning the complete pipeline.

---

## 6. URL Normalization

The same page can appear multiple times with slightly different URLs.

Examples:

- tracking parameters
- UTM parameters
- URL fragments
- trailing slashes
- duplicated query parameters

Before fetching and processing pages, URLs should be normalized where possible.

The system should preserve the original URL as well.

Important information:

- original URL
- normalized URL
- source
- discovered time

URL normalization helps reduce unnecessary fetching and duplicate processing.

---

## 7. Page Fetching

The fetcher retrieves the publicly accessible page.

Responsibilities include:

- HTTP requests
- redirects
- timeout handling
- retry handling
- backoff
- content-type detection
- HTTP status handling
- response size limits
- fetch timestamp
- failure recording

Example outcomes:

- 200 → process page
- 301/302 → follow redirect when allowed
- 404 → page unavailable
- 403 → access denied
- timeout → retry according to policy
- unsupported content → skip or route to appropriate parser

One failed URL must not stop the complete discovery task.

---

## 8. Public Access Rules

The platform should only process content that it is permitted to access.

The system must:

- respect robots.txt where applicable
- respect website terms and access restrictions
- use documented public APIs when available
- avoid login-protected content
- avoid bypassing CAPTCHA
- avoid bypassing authentication
- avoid bypassing technical access controls

The goal is reliable public-web discovery, not aggressive crawling.

---

## 9. Raw Document

The fetched response should be represented internally as a raw document before extracting normalized job information.

Possible fields:

- source URL
- normalized URL
- source name
- raw content
- content type
- HTTP status
- content hash
- discovered at
- fetched at

The raw document provides traceability.

It also allows extraction logic to evolve without losing the original fetched data.

---

## 10. Content Cleaning

Raw HTML usually contains a large amount of irrelevant information.

The processing layer should remove or reduce:

- scripts
- styles
- navigation
- advertisements
- repeated headers
- footers
- unrelated widgets

The objective is to create useful page text while preserving important job information.

Important content should include:

- job title
- company
- description
- location
- date
- time
- venue
- eligibility
- experience
- skills
- application instructions

---

## 11. Structured Data Extraction

Before relying heavily on plain text, the system should inspect structured data available on the page.

Possible sources:

- JSON-LD
- JobPosting structured data
- OpenGraph metadata
- HTML metadata

For example, a page may expose:

- job title
- hiring organization
- job location
- employment type
- description
- date posted

Structured data is generally easier to parse than arbitrary page text.

However, structured data should not automatically be trusted.

It must still pass validation.

---

## 12. Walk-in Detection

This is one of the most important stages of V1.

The system must determine whether a discovered page actually represents a walk-in opportunity.

Possible classification:

- WALK_IN
- NOT_WALK_IN
- UNCERTAIN

Signals may include phrases such as:

- walk-in interview
- walk in interview
- walk-in drive
- walkin drive
- walk-in recruitment
- walk-in hiring
- attend interview
- interview venue
- interview date
- reporting time
- venue address

The system should consider multiple signals rather than relying on a single keyword.

For example, a page containing the word "walk" should not automatically be classified as a walk-in job.

---

## 13. Job Extraction

Once a page is considered potentially relevant, the system extracts job information.

Possible fields:

- company
- job title
- description
- city
- country
- job type
- event date
- event start time
- event end time
- venue
- minimum experience
- maximum experience
- required skills
- eligibility
- application method
- application URL
- source URL
- source name

Extraction should preserve missing information rather than inventing values.

If the source does not provide a venue, the system should not generate one.

---

## 14. Extraction Priority

The preferred extraction order is:

1. Structured data
2. Source-specific parser
3. DOM-based extraction
4. Text pattern extraction
5. Optional AI-assisted extraction

Deterministic extraction should be preferred where practical.

AI can be introduced later for difficult or ambiguous pages.

AI output must still pass validation.

AI should not be treated as a source of truth.

---

## 15. Normalization

Different sources may represent the same information differently.

The normalization stage converts extracted information into a consistent internal format.

Examples:

Bangalore
→ Bengaluru

3-5 years
→ minimumExperience = 3
→ maximumExperience = 5

3+ years
→ minimumExperience = 3

Walk in
→ WALK_IN

Java / Spring Boot / Microservices
→ normalized skill list

Normalization makes downstream comparison and ranking easier.

---

## 16. Location Normalization

Location should be normalized carefully.

Examples:

- Bangalore → Bengaluru
- Bengaluru → Bengaluru
- Gurgaon → Gurugram

The system should preserve the original source value as well.

Possible model:

originalLocation:
Bangalore

normalizedCity:
Bengaluru

This keeps the original source information available for transparency.

---

## 17. Experience Normalization

Experience requirements can appear in many forms.

Examples:

"2-5 years"

"2 to 5 years"

"3+ years"

"Minimum 3 years"

"Freshers"

These should be converted into a consistent internal representation.

For example:

minimumExperience:
2

maximumExperience:
5

Unknown values should remain unknown rather than being guessed.

---

## 18. Validation

Extraction does not mean that a result is valid.

Every candidate opportunity should pass validation.

Minimum validation should check:

- company exists
- job title exists
- source URL exists
- walk-in evidence exists
- location is available when required
- event date is available or clearly inferable from the source
- event is not already expired
- extracted data is internally consistent

Additional validation can check:

- experience compatibility
- technology relevance
- application information
- venue
- interview timing

Invalid or incomplete results should not automatically be shown as high-quality opportunities.

---

## 19. Validation Confidence

The system should distinguish between:

Extraction Confidence

and

Verification / Trust

A page can be easy to parse but still contain unreliable information.

For example:

Extraction confidence:
High

Source verification:
Medium

These should remain separate concepts.

The system should not present an unverified listing as officially verified.

---

## 20. Deduplication

The public web frequently contains the same job on multiple pages.

For example:

- company website
- job board
- aggregator
- recruitment blog
- social post

The system should identify likely duplicates.

Potential matching signals:

- company
- normalized job title
- city
- event date
- venue
- application URL
- source URL
- description similarity

A duplicate should normally become another source reference for the same opportunity instead of another result.

---

## 21. Source Linking

One opportunity can have multiple sources.

Example:

Opportunity:
Java Backend Developer Walk-in

Sources:

- Company career page
- Job board
- Recruitment website

The system should retain these relationships.

The original or strongest source should be preferred when possible.

Users should be able to see where the opportunity came from.

---

## 22. Freshness

Freshness is a first-class property.

A walk-in opportunity can become useless once its event date has passed.

Possible freshness states:

- ACTIVE
- AGING
- EXPIRED

Example:

Event in 10 days:
ACTIVE

Event tomorrow:
ACTIVE

Event today:
ACTIVE, depending on event time

Event already finished:
EXPIRED

The system should avoid showing expired opportunities as active results.

---

## 23. Freshness Refresh

Some information can change after the first discovery.

Examples:

- interview date changed
- venue changed
- application link changed
- hiring drive cancelled
- job removed
- event postponed

The architecture should support refreshing important pages later.

V1 can use simple refresh mechanisms.

More advanced change detection can be added later.

---

## 24. Ranking

After validation and deduplication, valid opportunities should be ranked.

Ranking signals can include:

- technology relevance
- role relevance
- city match
- experience match
- event date
- freshness
- source quality
- completeness
- extraction confidence
- verification signals

The goal is not to show the largest number of results.

The goal is to show the most useful results first.

---

## 25. Search Relevance

For example, if the user searches:

Java + Bengaluru + 2-5 years

A result for:

Java Backend Developer
Bengaluru
3 years
Walk-in tomorrow

should rank higher than:

Java Developer
Hyderabad
8 years
Walk-in next month

Even if both pages were discovered successfully.

Relevance should be calculated from the user's search context.

---

## 26. Persistence

After processing, normalized opportunities should be stored in PostgreSQL.

The database becomes the source of truth for:

- opportunities
- companies
- sources
- discovery tasks
- freshness state
- normalized job information
- relationships between opportunities and sources

Raw documents may initially be stored in the database or another suitable storage layer depending on implementation.

As scale increases, large raw documents can move to object storage.

---

## 27. Discovery Task

A user search should create a discovery task rather than blocking the HTTP request until the complete web search finishes.

Conceptually:

Search Request
→ Discovery Task
→ Background Processing
→ Results

Possible task states:

- CREATED
- RUNNING
- COMPLETED
- PARTIALLY_COMPLETED
- FAILED

The task can also track:

- queries generated
- URLs discovered
- URLs fetched
- pages failed
- opportunities extracted
- opportunities rejected
- duplicates removed
- final opportunities stored

This provides visibility into pipeline performance.

---

## 28. Asynchronous Processing

Discovery is external-I/O-heavy and may take time.

The backend should therefore process discovery asynchronously.

V1 can use:

- Spring Async
- controlled thread pools
- bounded concurrency
- timeouts
- retries

A queue is not required in the first version.

A queue can be introduced later when discovery volume justifies it.

---

## 29. Failure Isolation

The pipeline should be designed so that individual failures do not stop the entire task.

Example:

100 URLs discovered

10 pages fail

90 pages continue through the pipeline.

Similarly:

1 parser fails

Other source parsers continue.

This is important because public web sources are unpredictable.

---

## 30. Rate Limiting and Backpressure

Different sources can have different limits.

The system should avoid uncontrolled concurrent requests.

Possible controls:

- per-source concurrency limits
- request rate limits
- timeouts
- retry limits
- exponential backoff
- response size limits

The goal is stable discovery rather than maximum request volume.

---

## 31. Idempotency

The same discovery request may run more than once.

The pipeline should avoid unnecessarily creating duplicate records.

Useful mechanisms include:

- normalized URLs
- content hashes
- source identifiers
- opportunity fingerprints
- database uniqueness constraints
- idempotent processing

Idempotency becomes increasingly important as background processing and scheduled refreshes are introduced.

---

## 32. Observability

Each major pipeline stage should produce useful logs and metrics.

Important metrics include:

- discovery task duration
- queries generated
- URLs discovered
- fetch success rate
- fetch failure rate
- extraction success rate
- walk-in classification counts
- validation rejection count
- duplicate count
- expired count
- opportunities created
- opportunities updated

This helps identify where the system is losing quality.

---

## 33. Source Quality

Not all sources should be treated equally.

The system can maintain source-level signals such as:

- source type
- historical success rate
- freshness
- extraction quality
- duplicate frequency
- availability
- trust level

For example:

Official company source
→ high source quality

Unknown aggregator
→ lower source quality

Source quality should influence ranking and trust signals, but should not blindly determine whether a result is valid.

---

## 34. Cost Awareness

The pipeline should track the cost of discovering useful opportunities.

A useful future metric is:

Cost per valid opportunity

This becomes important when external search APIs, AI extraction, or paid data providers are introduced.

The system should avoid spending expensive resources on obviously irrelevant pages.

For example:

Walk-in detection can happen before expensive AI extraction.

---

## 35. Deterministic First, AI When Valuable

The initial pipeline should prefer deterministic processing.

Example:

Search
→ Fetch
→ Structured Data
→ Rules / Parsers
→ Normalize
→ Validate

AI can be added for:

- ambiguous walk-in detection
- difficult page extraction
- semantic job matching
- duplicate similarity
- classification
- missing-field interpretation

AI should improve the pipeline, not become a mandatory dependency for every page.

---

## 36. Pipeline Data Separation

The system should conceptually separate:

Raw Data
→ Extracted Data
→ Normalized Data
→ Validated Opportunity

This prevents extraction changes from destroying source information.

Example:

Raw source says:

"Walk in interview at our Bangalore office"

Extracted:

city = Bangalore

Normalized:

city = Bengaluru

Validated:

valid walk-in opportunity

Each layer has a different responsibility.

---

## 37. Frontend Boundary

The frontend does not perform discovery.

The frontend is responsible for:

- search form
- loading state
- discovery status
- result listing
- filtering
- opportunity details
- source links
- error handling

The backend is responsible for:

- search
- discovery
- fetching
- extraction
- normalization
- validation
- deduplication
- freshness
- ranking
- persistence

This keeps the system secure, testable, and scalable.

The frontend application exists in the repository from Day 1, but backend discovery and APIs are the primary implementation focus initially.

Frontend implementation begins after the backend pipeline and APIs are reliable enough to consume.

---

## 38. V1 Pipeline

The initial production-oriented pipeline should be:

1. Receive search request
2. Validate request
3. Create SearchContext
4. Create discovery task
5. Generate search queries
6. Search configured public sources
7. Collect candidate URLs
8. Normalize URLs
9. Remove obvious duplicate URLs
10. Fetch pages
11. Store raw document information
12. Clean page content
13. Read structured data
14. Detect walk-in signals
15. Extract job information
16. Normalize fields
17. Validate opportunity
18. Deduplicate opportunities
19. Calculate freshness
20. Calculate relevance
21. Store valid opportunities
22. Expose results through API
23. Frontend displays results

This is the core V1 discovery engine.

---

## 39. Future Pipeline Improvements

The architecture should allow future additions such as:

- more search providers
- direct company career-page crawling
- ATS integrations
- scheduled discovery
- scheduled refresh
- change detection
- source quality scoring
- semantic matching
- AI-assisted extraction
- AI-assisted classification
- better duplicate detection
- search index
- Redis caching
- distributed workers
- queue-based processing
- object storage for raw documents
- personalized recommendations

These should be introduced when actual product requirements justify them.

---

## 40. Scalability Direction

V1 should remain a modular monolith.

Initial architecture:

Frontend
→ Spring Boot Backend
→ PostgreSQL

Backend discovery:

Discovery Task
→ Controlled Async Workers
→ External Sources
→ Processing Pipeline
→ PostgreSQL

As volume increases, the architecture can evolve toward:

Frontend
→ Load Balancer
→ Multiple API Instances
→ Discovery Queue
→ Discovery Workers
→ PostgreSQL
→ Redis
→ Search Index
→ Object Storage

The pipeline stages should remain logically separated even if they initially run inside one application.

---

## 41. Core Design Principles

The data pipeline should follow these principles:

### 1. Relevance over quantity

A smaller set of useful jobs is better than hundreds of irrelevant results.

### 2. Freshness over volume

An expired walk-in opportunity has little value.

### 3. Trust over hype

Always preserve the source and avoid presenting assumptions as facts.

### 4. Never invent missing information

Unknown data should remain unknown.

### 5. Raw data and normalized data are different

Keep source information separate from processed information.

### 6. One failure should not stop discovery

Public web sources will fail.

### 7. Deduplicate before showing results

Users should not see the same opportunity repeatedly.

### 8. Source transparency is mandatory

Users should be able to identify where an opportunity came from.

### 9. Deterministic processing first

Use rules and structured data wherever possible.

### 10. Scale only when required

Do not introduce Kafka, Kubernetes, microservices, or a search engine before the product needs them.

---

## Final Pipeline

The final conceptual pipeline for V1 is:

User Search
→ Search Context
→ Discovery Task
→ Query Generation
→ Source Discovery
→ URL Normalization
→ Page Fetching
→ Raw Document
→ Content Processing
→ Structured Data Extraction
→ Walk-in Detection
→ Job Extraction
→ Normalization
→ Validation
→ Opportunity Deduplication
→ Freshness
→ Ranking
→ PostgreSQL
→ Search API
→ Frontend

The most important responsibility of this pipeline is simple:

**Find relevant upcoming walk-in opportunities from the public web, remove noise and duplicates, keep results fresh, and show users the original source.**
