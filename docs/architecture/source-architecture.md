# Source Architecture

## Purpose

The Job Discovery Platform depends on multiple public sources to discover job opportunities.

No single website or provider should be treated as the complete source of job information.

The source architecture is therefore designed around:

- Multiple source types
- Source independence
- Pluggable adapters
- Source-specific logic isolation
- Public access
- Rate limiting
- Failure isolation
- Source transparency
- Source quality
- Future extensibility

The discovery pipeline should work even when one source becomes unavailable.

---

## 1. Source Architecture Overview

The source flow is:

Search Context
→ Query Generation
→ Source Selection
→ Source Adapter
→ Candidate URLs
→ URL Normalization
→ Page Fetching
→ Processing Pipeline

The rest of the pipeline should not need to know how a URL was discovered.

---

## 2. Source Types

The platform can discover opportunities from several types of sources.

Initial and future source categories include:

- Search engines
- Search APIs
- Company career pages
- Public ATS
- Job boards
- Job aggregators
- Public APIs
- RSS/XML feeds
- Sitemaps
- Other publicly accessible sources

Each source type has different discovery and access characteristics.

---

## 3. Search Providers

Search providers are useful for broad discovery.

Example query:

"walk-in interview" "Bengaluru" "Java"

The provider returns candidate pages.

The platform then fetches and validates those pages independently.

The search provider is therefore a discovery mechanism, not the final source of truth.

---

## 4. Search Provider Abstraction

Search providers should be isolated behind an interface.

Conceptually:

SearchProvider

Responsibilities:

- accept search query
- execute search
- return candidate results
- provide source metadata
- handle provider-specific behaviour

Example implementations:

- SearchProviderAdapter
- SearchApiAdapter
- GoogleJobsAdapter

The exact providers can change without changing the core discovery pipeline.

---

## 5. Job Source Abstraction

Direct job sources should also use a common abstraction.

Conceptually:

JobSource

Responsibilities may include:

- discovering jobs
- returning candidate URLs
- identifying source
- handling source-specific parsing
- exposing source metadata

This keeps source-specific logic outside the central pipeline.

---

## 6. Source Adapter

Each external source should have its own adapter when special handling is required.

Examples:

SearchEngineAdapter

GreenhouseAdapter

LeverAdapter

CompanyCareerPageAdapter

JobBoardAdapter

The adapter should translate source-specific information into the platform's internal format.

The rest of the backend should not depend on source-specific response structures.

---

## 7. Source Adapter Responsibility

An adapter can be responsible for:

- request construction
- source-specific parameters
- response parsing
- URL extraction
- source metadata
- source-specific pagination
- source-specific error handling

An adapter should not be responsible for:

- final opportunity validation
- global deduplication
- global ranking
- freshness calculation
- frontend formatting

Those belong to other backend modules.

---

## 8. Source Independence

The platform should not become dependent on one provider.

For example:

If Search Provider A becomes unavailable:

Search Provider B
→ continues discovery

If one job board blocks requests:

Other sources
→ continue discovery

If a source changes its HTML:

Only the relevant adapter/parser should need modification where applicable.

This is important for long-term reliability.

---

## 9. Source Discovery Strategy

The discovery engine should combine multiple strategies.

### Strategy 1: Search Provider Discovery

Generate search queries and use search providers to discover candidate URLs.

### Strategy 2: Direct Public Source Discovery

Where a public source has a documented API, ATS endpoint, feed, sitemap, or accessible listing page, the platform can use it directly.

### Strategy 3: Company Career Pages

For future company-specific discovery, public company career pages can become important direct sources.

The platform should choose sources based on the search context.

---

## 10. Source Selection

Not every source needs to be queried for every search.

Source selection can consider:

- city
- role
- technology
- company
- opportunity type
- source capabilities
- historical source quality
- source availability
- cost
- rate limits

For V1, source selection can remain simple.

For example:

Walk-in + Java + Bengaluru

may use:

- search provider
- selected job boards
- selected public sources

More advanced source routing can be introduced later.

---

## 11. Query Generation and Sources

Query generation should remain separate from source execution.

Example:

SearchContext:

Java
Bengaluru
Walk-in

Query Generator produces:

"walk-in interview" "Bengaluru" "Java"

"walk-in drive" "Bengaluru" "Java"

The source adapter receives the query.

This separation allows the same query generation logic to work with multiple search providers.

---

## 12. Candidate URL Model

The source discovery stage should return a common candidate representation.

Possible fields:

- url
- normalizedUrl
- sourceId
- sourceType
- title
- snippet
- discoveredAt

The title and snippet are optional.

They can help prioritize fetching.

The URL remains the primary input to the page-fetching stage.

---

## 13. URL Normalization

Source adapters should return the URL they discovered.

A central URL normalization stage should then:

- normalize URL
- remove unnecessary tracking parameters where safe
- normalize fragments
- identify obvious duplicates

The central pipeline should own global URL normalization rather than implementing it differently in every adapter.

---

## 14. Source Metadata

Every discovered URL should retain source information.

Example:

source:
Search Provider A

sourceType:
SEARCH_ENGINE

sourceUrl:
original page URL

discoveredAt:
timestamp

This makes source tracing possible.

---

## 15. Original Source vs Discovery Source

The platform should distinguish between:

Discovery Source

and

Original Job Source

Example:

A search engine discovers:

example.com/jobs/java-walkin

The search engine is the discovery source.

The webpage itself is the original source.

The user should ultimately be able to open the original job source.

This distinction is important for transparency.

---

## 16. Source Quality

Each source can have a quality signal.

Possible factors:

- historical validity rate
- freshness
- extraction success
- duplicate frequency
- source stability
- availability
- official status
- completeness of job information

Example:

Official company career page
→ strong source quality

Unknown aggregator
→ lower source quality

Source quality should influence ranking and trust.

It should not automatically determine whether a result is valid.

---

## 17. Source Verification

Source quality and verification are different concepts.

Source quality:

How reliable has this source historically been?

Verification:

Has this particular opportunity been sufficiently confirmed?

A high-quality source can still contain an outdated listing.

Therefore, the opportunity itself needs freshness and validation.

---

## 18. Source Health

The backend should track source health signals.

Possible metrics:

- request success rate
- response time
- HTTP failure rate
- extraction success rate
- number of valid opportunities
- duplicate rate
- last successful fetch
- last failure

This can help determine whether a source should continue receiving traffic.

---

## 19. Source Failure Handling

A source failure should not stop the discovery task.

Example:

Source A:
FAILED

Source B:
SUCCESS

Source C:
SUCCESS

The discovery task continues.

If enough useful results are found, the task can complete as:

COMPLETED

If important portions failed but results are still available:

PARTIALLY_COMPLETED

---

## 20. Retries

Retries should be controlled.

Possible retry conditions:

- temporary network error
- timeout
- temporary server error
- rate-limit response where retry is appropriate

Do not repeatedly retry:

- permanent 404
- blocked resource
- invalid URL
- unsupported content

Use:

- retry limit
- exponential backoff
- source-specific limits

---

## 21. Rate Limiting

Different sources may impose different limits.

The platform should maintain source-specific controls.

Possible controls:

- requests per second
- requests per minute
- maximum concurrent requests
- daily limits
- monthly provider limits

The system should avoid uncontrolled crawling.

---

## 22. Public Access Rules

Only appropriately public and permitted sources should be processed.

The platform must:

- respect robots.txt where applicable
- respect website terms and access restrictions
- use documented APIs where available
- avoid login-protected content
- avoid CAPTCHA bypass
- avoid authentication bypass
- avoid bypassing technical restrictions

If a source cannot be accessed appropriately, the platform should skip it rather than attempt to circumvent the restriction.

---

## 23. Search API Usage

External search APIs may have:

- API keys
- request limits
- usage costs
- daily limits
- monthly limits
- commercial-use restrictions

These details should remain in configuration rather than being hardcoded into business logic.

Example configuration concepts:

search.provider.enabled

search.provider.api-key

search.provider.timeout

search.provider.rate-limit

The actual provider configuration should be kept outside source code where appropriate.

---

## 24. Direct Public APIs

Some job platforms expose public APIs.

Examples can include:

- Public ATS APIs
- Company job-board APIs
- Public job APIs

A dedicated adapter should be created for each integration that provides enough value.

The platform should use documented APIs according to their terms.

---

## 25. Greenhouse and Lever

Public ATS integrations can become useful direct sources.

For example:

Greenhouse public job board endpoints can expose published company jobs.

Lever also provides public postings for supported company sites.

These sources are especially useful for future regular-job and company-specific discovery.

They may not be the primary mechanism for V1 walk-in discovery.

The architecture should nevertheless allow them to be added without changing the core pipeline.

---

## 26. Company Career Pages

Company career pages can be important high-quality sources.

They may contain:

- open positions
- hiring drives
- interview information
- application links
- job descriptions

Career pages often differ significantly in structure.

Therefore, the system should initially use generic extraction where possible and introduce source-specific parsers only when justified.

---

## 27. Generic vs Source-Specific Parsing

The system should not create a custom parser for every website from the beginning.

Preferred approach:

Generic extraction
→ works for many pages

Source-specific parser
→ added when a source is important and generic extraction is insufficient

This prevents the codebase from becoming a collection of fragile website-specific parsers.

---

## 28. Source Parser Priority

For a known source:

Source-specific parser
→ preferred

For an unknown source:

Generic extraction
→ used

If structured JobPosting data exists:

Structured extraction
→ preferred

The final extracted result still passes through the common normalization and validation pipeline.

---

## 29. Source-Specific Configuration

Not every difference requires custom code.

Some source behaviour can be configuration-driven.

Possible configuration:

- base URL
- source type
- pagination rules
- allowed domains
- rate limit
- parser type
- timeout
- enabled/disabled status

This can reduce unnecessary source-specific code.

---

## 30. Source Registration

A source should be registered before the platform uses it.

Possible information:

- name
- source type
- base URL
- adapter
- active status
- quality score
- rate limit
- configuration

This allows the backend to manage sources consistently.

---

## 31. Source Enable / Disable

A source should be possible to disable without changing the application code.

Reasons may include:

- source outage
- API quota exhausted
- parser broken
- source quality dropped
- temporary maintenance
- terms/access changes

The discovery engine should skip disabled sources.

---

## 32. Source Priority

Sources can have a priority.

Example:

Priority 1:
Official company source

Priority 2:
Public ATS

Priority 3:
Major job board

Priority 4:
Aggregator

Priority 5:
Other public pages

Priority should influence discovery order and ranking, but should not prevent useful lower-priority sources from being used.

---

## 33. Source Cost

Some sources may be free.

Others may have usage-based costs.

The source model should therefore allow cost awareness.

Possible source properties:

- free
- paid
- estimated cost
- quota
- current usage

V1 can keep this simple.

Cost-aware source selection becomes more important when discovery volume grows.

---

## 34. Source Coverage

Source quality is not the same as source coverage.

A source may be highly reliable but contain very few opportunities.

Another source may have broad coverage but more noise.

The system should consider both:

Quality
and
Coverage

This helps the platform balance reliable sources with broad discovery.

---

## 35. Source Freshness

Sources can have different freshness characteristics.

For example:

- frequently updated career page
- daily job board
- old recruitment article

The system should consider source freshness when deciding how valuable a discovered result may be.

However, final opportunity freshness should be determined from the opportunity itself.

---

## 36. Source Deduplication

The same opportunity may be discovered through multiple sources.

Example:

Search Provider
→ Job Board
→ Company Career Page

The source architecture should preserve all discovery references.

Global opportunity deduplication happens later in the pipeline.

Source adapters should not attempt to perform global deduplication.

---

## 37. Source Data Flow

The source layer should follow:

SearchContext
→ Source Selection
→ Query
→ Source Adapter
→ Candidate URLs
→ Common Candidate Model
→ URL Normalization
→ Fetching
→ Extraction

The source layer ends after providing candidates.

The rest of the pipeline remains source-independent.

---

## 38. Source Adapter Interface

Conceptually, the backend may define an interface similar to:

JobSource

Possible methods:

- getSourceName()
- getSourceType()
- supports(SearchContext)
- discover(SearchContext)
- healthCheck()

The exact Java interface will be finalized during implementation.

The interface should remain small.

Do not put extraction, ranking, persistence, or frontend behaviour into the source interface.

---

## 39. Source Adapter Package

A possible backend structure is:

discovery/
    source/
        JobSource
        SearchProvider
        SearchEngineAdapter
        GreenhouseAdapter
        LeverAdapter
        CompanyCareerPageAdapter

The exact package names can evolve.

The important rule is to keep external-source logic isolated.

---

## 40. Source Configuration

Configuration should support:

- enabled/disabled
- timeout
- rate limit
- concurrency
- API credentials where required
- source-specific settings

Secrets such as API keys must not be committed to GitHub.

Use environment variables or a secure secret-management mechanism.

---

## 41. Source Observability

Each source should be measurable.

Useful metrics:

source.requests

source.success

source.failure

source.timeout

source.rate_limit

source.urls_discovered

source.pages_fetched

source.valid_opportunities

source.duplicates

source.extraction_failures

These metrics help determine which sources actually create product value.

---

## 42. Source Quality Feedback Loop

The source system should eventually learn from actual outcomes.

Example:

Source A:

1000 pages
→ 100 valid opportunities

Source B:

1000 pages
→ 5 valid opportunities

Source A has better effective yield.

The system can use this information to improve source priority.

A useful future metric is:

Valid Opportunities / Pages Processed

This is more meaningful than raw page count.

---

## 43. Source Reliability Feedback

Similarly:

Source A:
99% fetch success

Source B:
40% fetch success

The platform can adjust source priority or temporarily disable unreliable sources.

This should happen carefully.

A temporary outage should not permanently reduce a source's value.

---

## 44. Adding a New Source

Adding a source should ideally require:

1. Register source
2. Implement adapter if required
3. Add source configuration
4. Add tests
5. Verify public access
6. Verify extraction
7. Verify deduplication
8. Verify source transparency
9. Enable source

The core discovery pipeline should remain unchanged.

---

## 45. V1 Source Strategy

V1 should not attempt to integrate every possible job website.

Start with a small set of useful sources.

Recommended initial approach:

1. Search provider discovery
2. Public pages discovered through search
3. Selected high-value direct sources where integration is straightforward

Measure:

- valid opportunities
- relevance
- freshness
- source reliability
- cost

Then add sources based on actual results.

---

## 46. Source Expansion

Future source expansion can include:

- More search providers
- More job boards
- More public ATS platforms
- Company career pages
- Public APIs
- RSS feeds
- Sitemaps
- Industry-specific sources
- Location-specific sources

Each addition should be evaluated based on product value.

---

## 47. Source Architecture and Scalability

The source layer should support independent scaling later.

Initial:

Spring Boot
→ Source Adapters
→ External Sources

Later:

Discovery Queue
→ Source Workers
→ External Sources

Different workers can process different source types.

For example:

Search Workers

ATS Workers

Career Page Workers

This can be introduced only when workload requires it.

---

## 48. Source Architecture and Security

Source credentials must never be stored in:

- GitHub
- source code
- frontend
- logs
- API responses

Use:

- environment variables
- secret management
- deployment configuration

The frontend should never receive external source API credentials.

All external provider communication should happen from the backend.

---

## 49. Source Architecture and Product Trust

The source architecture directly supports product trust.

For every opportunity, the platform should be able to answer:

Where did this come from?

When was it discovered?

When was it last seen?

What source provided it?

Is it still active?

Was it duplicated from another source?

This information should be available internally and appropriate portions should be exposed to users.

---

## 50. Final V1 Source Architecture

The V1 source architecture is:

Search Context
|
v
Query Generator
|
v
Source Selection
|
+----------------------+
|                      |
v                      v
Search Providers       Direct Public Sources
|                      |
+----------+-----------+
           |
           v
    Source Adapters
           |
           v
    Candidate URLs
           |
           v
    URL Normalization
           |
           v
     Page Fetching
           |
           v
    Common Processing
           |
           v
Extraction
→ Normalization
→ Validation
→ Deduplication
→ Freshness
→ Ranking
→ PostgreSQL

The key architectural rule is:

**The discovery pipeline must depend on a source abstraction, not on individual websites.**

This allows the platform to add, remove, replace, or scale sources without redesigning the core product.
