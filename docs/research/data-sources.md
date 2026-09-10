# Data Sources Research

## 1. Purpose

This document evaluates the possible data sources for the Job Discovery Platform.

The goal is to understand:

- Where job opportunities can be discovered
- Which sources provide APIs
- Which sources provide structured data
- Which sources are useful for walk-in opportunities
- Which sources are useful for regular jobs
- Source quality
- Freshness
- Coverage
- Cost and limits
- Access restrictions
- Technical integration effort
- Long-term product fit

This document is specifically for product decisions.

We should not choose a data source only because it is technically easy to integrate.

The source should be evaluated based on the quality of opportunities it can help us discover.

---

# 2. Important Product Decision

There should not be one single data source.

No single API or website can reliably provide every relevant job opportunity across the public internet.

Therefore the platform should use a source-independent architecture.

Conceptually:

    Multiple Sources
          ↓
    Discovery Layer
          ↓
    Common Pipeline
          ↓
    Normalized Opportunity
          ↓
    Validation
          ↓
    Deduplication
          ↓
    Freshness
          ↓
    Ranking

The source layer should be replaceable.

---

# 3. Source Categories

Potential sources can be divided into:

1. Search engines / search APIs
2. Public job APIs
3. Company career pages
4. Public ATS APIs
5. Job boards
6. Job aggregators
7. Structured data embedded in webpages
8. RSS/XML feeds
9. Direct employer hiring pages
10. Other publicly accessible pages

Each category has different strengths.

---

# 4. Search Engine Discovery

## Description

Search engines can discover webpages that contain job and walk-in information.

Example query:

    "walk-in interview" "Bengaluru" "Java"

Other query variations:

    "walk-in drive" "Bengaluru" "Java"
    "walkin interview" "Bengaluru" "Java"
    "walk-in recruitment" "Bengaluru" "Java"
    "Java developer" "walk-in" "Bengaluru"

The search engine is used to discover URLs.

The original webpage remains the source that should be fetched and processed.

---

## Advantages

- Broad discovery
- Can discover unknown websites
- Useful for walk-in opportunities
- Does not require maintaining a list of every job website
- Useful when no API exists for a source

---

## Limitations

Search engines do not provide guaranteed complete coverage of the public web.

A page may:

- Not be indexed
- Be newly published
- Be removed
- Be blocked from crawling
- Be poorly structured
- Be difficult to discover

Therefore search should be treated as a discovery mechanism rather than a complete job database.

---

## Product Fit

Very high for V1.

The initial product specifically needs to discover walk-in pages across many different websites.

---

# 5. Google JobPosting Structured Data

Google supports `JobPosting` structured data for individual job posting pages.

Source:

https://developers.google.com/search/docs/appearance/structured-data/job-posting

Google states that `JobPosting` structured data can help job postings appear in Google's job-search experience.

The structured data can contain information such as:

- Job title
- Description
- Posting date
- Hiring organization
- Location
- Employment information
- Experience requirements
- Other job properties

---

## Why It Matters To Us

Some job pages may contain structured job information even when the visible HTML is difficult to parse.

Therefore the extraction pipeline should inspect structured data before relying entirely on generic HTML parsing.

Recommended extraction order:

    Structured Data
          ↓
    Known Source Parser
          ↓
    HTML / DOM Extraction
          ↓
    Text Extraction
          ↓
    AI-assisted Extraction
          ↓
    Validation

AI should not be the first extraction mechanism when deterministic structured information is already available.

---

## Limitations

Not every job page contains JobPosting structured data.

Even when structured data exists:

- It may be incomplete.
- It may be stale.
- It may not contain walk-in-specific event information.
- It still needs validation against the page.

Therefore structured data is an extraction signal, not automatic proof of correctness.

---

# 6. Adzuna API

## Source

Adzuna provides a REST API for job advertisements.

Official documentation:

https://developer.adzuna.com/overview

The API provides search endpoints for job listings.

The API requires:

- app_id
- app_key

The search API supports job advertisement queries using parameters such as keywords and locations.

Source:

https://developer.adzuna.com/docs/search

---

## What It Can Provide

Adzuna can provide structured job listing information.

The API is useful for:

- Keyword-based job search
- Location-based search
- Job listing discovery
- Standardized job data

---

## Access Limits

Adzuna documents default API limits including:

- 25 requests per minute
- 250 requests per day
- 1000 requests per week
- 2500 requests per month

Source:

https://developer.adzuna.com/docs/terms_of_service

Adzuna also states that commercial use beyond the trial period is subject to its terms and access arrangements.

---

## Advantages

- Structured API
- Easy machine-readable responses
- Keyword search
- Location search
- Large job database
- Useful for regular jobs

---

## Limitations

The API represents Adzuna's own job-advertisement dataset.

It does not provide a complete representation of every public job opportunity on the internet.

It is also not specifically designed around walk-in events.

Therefore:

    Adzuna ≠ Complete Discovery Engine

It should be treated as one source.

---

## Product Fit

### V1 Walk-ins

Medium to low.

### Future regular jobs

High.

### Future company/job search

Potentially useful.

---

# 7. Greenhouse Job Board API

## Source

Greenhouse provides a Job Board API.

Official documentation:

https://docs.greenhouse.io/job-board.html

Greenhouse states that public Job Board data is available without authentication for GET endpoints.

The API can return published job information.

---

## Available Information

The job response can include information such as:

- Job ID
- Job title
- Updated time
- Requisition ID
- Location
- Public job URL
- Job description
- Departments
- Offices
- Metadata

The API supports:

    GET /v1/boards/{board_token}/jobs

and individual job retrieval.

---

## Advantages

- Official ATS source
- Structured JSON
- Public job-board data
- Job descriptions available
- Location information
- Updated timestamps
- Direct source URL
- High source quality

---

## Limitations

The API is company-specific.

We need to know the company's Greenhouse board token.

It is not a global search API across all companies.

It primarily provides regular job postings.

It is not designed specifically for walk-in events.

---

## Product Fit

### V1 Walk-ins

Low to medium.

### Future regular jobs

Very high.

### Company-specific discovery

Very high.

---

# 8. Lever Postings API

## Source

Lever provides a public Postings API.

Official documentation:

https://hire.lever.co/developer/documentation

Documentation repository:

https://github.com/lever/postings-api

Lever states that published job postings are publicly viewable through its hosted job sites.

The public postings API can retrieve published postings.

---

## Available Information

The API supports:

- List job postings
- Retrieve individual job postings
- Query postings
- Retrieve structured posting data

Published postings can be retrieved through endpoints such as:

    GET /v0/postings/SITE

---

## Important Limitation

Lever states that its Postings API does not provide full-text search across all open jobs.

The API is primarily scoped around a company's posting site.

Therefore it is not a universal job-search API.

---

## Advantages

- Official ATS data
- Structured responses
- Public published jobs
- Direct company source
- Good source quality
- Useful for company-specific discovery

---

## Limitations

- Company-specific
- Requires knowing the site's identifier
- Mainly regular jobs
- Not specifically designed for walk-ins

---

## Product Fit

### V1 Walk-ins

Low.

### Future regular jobs

Very high.

### Company-specific discovery

Very high.

---

# 9. Search APIs / Google Jobs Providers

A third-party provider such as SerpApi provides access to Google Jobs search results through an API.

Source:

https://serpapi.com/google-jobs-api

The API supports:

- Search query
- Location
- Pagination
- Structured job results

The endpoint uses:

    engine=google_jobs

The API returns structured job results in JSON.

---

## Advantages

- Broad job discovery
- Search-based
- Location-aware
- Structured output
- Faster to integrate than implementing our own search-engine extraction

---

## Limitations

- Third-party dependency
- Cost
- Usage limits
- Search coverage depends on Google Jobs
- Not specifically a walk-in database
- Search results can change

SerpApi is therefore an infrastructure option rather than the product itself.

---

## Product Fit

### V1 Walk-ins

Potentially high for discovery experiments.

### Future regular jobs

High.

### Long-term

Useful as one source, but should not be the only source.

---

# 10. Google Jobs Listing Results

SerpApi also provides a separate Google Jobs Listing API.

Source:

https://serpapi.com/google-jobs-listing-api

It can retrieve more detail for a specific Google Jobs result using its job ID.

However, the available fields can change as Google changes its Jobs interface.

Therefore we should not design our core data model around provider-specific response fields.

Instead:

    Provider Response
          ↓
    Provider Adapter
          ↓
    Common Job Model

---

# 11. Company Career Pages

Company career pages are among the most valuable sources for trust.

Example:

    company.com/careers

The company itself controls the information.

Potential information:

- Role
- Location
- Experience
- Skills
- Job description
- Posting date
- Application URL
- Employment type

---

## Advantages

- High source quality
- Direct employer source
- Lower risk of copied information
- Useful for company-specific discovery

---

## Limitations

- Every company uses different website structures
- Some pages use ATS systems
- Some use JavaScript-heavy pages
- Some do not expose structured APIs
- Discovery coverage is difficult at scale
- Walk-in information may not always be published on the career page

---

## Product Fit

### V1 Walk-ins

Medium.

### Future regular jobs

Very high.

### Trust

Very high.

---

# 12. Public ATS Sources

Applicant Tracking Systems can expose structured public job postings.

Examples include:

- Greenhouse
- Lever
- Other public ATS systems

The important product advantage is that ATS data is generally more structured than arbitrary webpages.

Possible architecture:

    Company
       ↓
    ATS
       ↓
    Public Job Posting
       ↓
    Source Adapter
       ↓
    Normalized Job

---

## Product Fit

### Walk-ins

Limited.

### Regular jobs

Very high.

### Company tracking

Very high.

---

# 13. Job Boards

Job boards can contain large numbers of job listings.

Potential examples include:

- General job portals
- Technology job boards
- Industry-specific job boards
- Recruitment websites

They can be useful for broad coverage.

However, direct automated access depends on the specific site's:

- Public APIs
- Terms
- Robots rules
- Access controls
- Technical architecture

The system should not assume that every job board can be crawled.

---

# 14. Job Aggregators

Aggregators can provide additional coverage.

However, they introduce another problem:

The same job may already exist on the original company site.

Therefore aggregator sources are useful for discovery but may have lower source authority than the original employer source.

The platform should retain the source hierarchy.

Example:

    Official Company Source
            ↓
    Public ATS
            ↓
    Established Job Platform
            ↓
    Other Public Source

This should influence ranking and trust.

---

# 15. RSS / XML Feeds

Some websites publish jobs through:

- RSS
- XML
- Atom
- Sitemaps

These formats can be easier to consume than HTML.

Advantages:

- Structured
- Lightweight
- Easy to poll
- Useful for change detection

Limitations:

- Not every site provides them
- Content may be incomplete
- Walk-in events may not be represented

---

# 16. Sitemaps

Some websites publish XML sitemaps containing URLs.

Sitemaps can help discover pages that may not be easy to find through navigation.

Google documents sitemaps as part of its crawling ecosystem.

Sitemaps can therefore be considered as an additional discovery signal when publicly available.

However:

    Sitemap URL ≠ Job URL

The system still needs to determine which URLs contain relevant opportunities.

---

# 17. Robots.txt

Websites can publish a `robots.txt` file that specifies crawler rules.

Google documents the Robots Exclusion Protocol and how its crawlers interpret robots.txt.

Sources:

https://developers.google.com/crawling/docs/robots-txt/robots-txt-spec

https://developers.google.com/crawling/docs/robots-txt/create-robots-txt

A robots.txt file can contain:

- User-agent
- Allow
- Disallow
- Sitemap

---

## Product Rule

Our system should respect applicable access restrictions and should not attempt to bypass them.

We should not:

- Bypass CAPTCHA
- Bypass authentication
- Circumvent access controls
- Evade anti-bot systems
- Access private information

If a source is not appropriately accessible, the system should use another source.

---

# 18. Direct Web Crawling

Direct crawling means:

    Source Website
          ↓
    HTTP Request
          ↓
    HTML
          ↓
    Extraction

This provides maximum control but also creates maximum maintenance.

Different websites have different:

- HTML structures
- JavaScript behavior
- Rate limits
- Access policies
- Page layouts

Therefore generic crawling should not be the only strategy.

---

# 19. Recommended Fetching Strategy

The platform should use a layered approach.

### Layer 1

API / structured source

### Layer 2

Known source parser

### Layer 3

Public webpage extraction

### Layer 4

Search-engine discovery

The actual order can differ depending on the source.

The important point is that the source-specific logic should stay behind an abstraction.

---

# 20. Source Adapter Architecture

Each source should be represented internally through an adapter.

Conceptually:

    JobSource
        |
        +-- SearchProviderSource
        +-- AdzunaSource
        +-- GreenhouseSource
        +-- LeverSource
        +-- CompanyCareerSource
        +-- RssSource
        +-- WebPageSource

Each source converts its native data into a common internal model.

Example:

    Source Data
         ↓
    Source Adapter
         ↓
    DiscoveredPage / RawJob
         ↓
    Common Pipeline

---

# 21. Source Metadata

Every source should have metadata.

Potential fields:

- sourceName
- sourceType
- baseUrl
- sourceQuality
- accessMethod
- lastCheckedAt
- enabled
- rateLimit
- notes

Example:

    sourceName: Greenhouse
    sourceType: ATS
    accessMethod: PUBLIC_API
    sourceQuality: HIGH

This allows the platform to manage sources independently.

---

# 22. Source Quality

A source-quality score can eventually consider:

- Official employer source
- Structured API
- Freshness
- Historical accuracy
- Completeness
- Duplicate frequency
- Availability
- Reliability

This score should not be treated as absolute truth.

It is only one ranking signal.

---

# 23. Freshness by Source

Different sources may need different refresh strategies.

Example:

### Walk-in page

Potentially check frequently because the event is time-sensitive.

### Company career page

Check periodically for new jobs.

### Historical page

May not need frequent checks.

The refresh strategy should eventually depend on:

- Opportunity type
- Event date
- Source behavior
- Last change
- Previous fetch result

---

# 24. Source Deduplication

The same source may return the same opportunity repeatedly.

Therefore source-level deduplication should happen before global opportunity deduplication where possible.

Example:

    Search Query A
         ↓
    Source URL X

    Search Query B
         ↓
    Source URL X

The system should avoid processing the same URL unnecessarily.

A normalized URL or URL fingerprint can help.

---

# 25. Global Opportunity Deduplication

After source-level deduplication, the system should compare opportunities across different sources.

Example:

    Company A
    Java Developer
    Bengaluru
    20 September

Source A:

    company.com/careers/123

Source B:

    jobsite.com/job/456

These may represent the same real-world opportunity.

The platform should attempt to merge them.

---

# 26. Data Source Comparison

| Source | V1 Walk-in | Future Jobs | Structured | Coverage | Source Quality | Integration |
|---|---:|---:|---:|---:|---:|---:|
| Search Provider | High | High | Medium/High | High | Depends on result | Medium |
| Google Jobs Provider | Medium/High | High | High | High | Depends on original source | Easy/Medium |
| Adzuna | Low/Medium | High | High | High | Medium/High | Easy |
| Greenhouse | Low | Very High | Very High | Company-specific | High | Easy |
| Lever | Low | Very High | Very High | Company-specific | High | Easy |
| Company Career Page | Medium | Very High | Low/Medium | Company-specific | Very High | Medium/Hard |
| Public ATS | Low | Very High | Very High | Company-specific | High | Medium |
| Job Board | Medium/High | High | Medium | High | Medium | Varies |
| Aggregator | High/Medium | High | Medium | High | Medium | Varies |
| RSS/XML | Medium | Medium/High | High | Low/Medium | Depends on source | Easy |
| Generic Web Page | High | High | Low | Very High | Depends on source | Hard |

The values above are product-level assessments, not guarantees.

Actual coverage and quality must be measured through implementation experiments.

---

# 27. Recommended V1 Data Sources

For the first working product, the recommended strategy is:

## Primary Discovery

Search-provider-based discovery.

Goal:

    User Search
       ↓
    Relevant URLs

This gives us broad discovery without maintaining hundreds of source integrations.

---

## Secondary Discovery

A small number of direct public sources.

These can be added after the basic search pipeline works.

Examples:

- Selected walk-in websites
- Selected public job sources
- Public APIs where useful

---

## Extraction

Use the original source page as the primary evidence.

Possible extraction layers:

    Structured Data
          ↓
    HTML
          ↓
    Text
          ↓
    AI only when necessary

---

# 28. Recommended Future Data Sources

Once regular job discovery is introduced, prioritize:

1. Company career pages
2. Public ATS systems
3. Greenhouse
4. Lever
5. Public job APIs
6. Search providers
7. Job boards
8. Aggregators

The exact priority should be based on actual coverage and user demand.

---

# 29. What Should Not Be a Core Dependency

The core platform should not depend entirely on:

- One search provider
- One job API
- One job board
- One ATS
- One scraping provider

Any external provider can:

- Change its API
- Change pricing
- Add rate limits
- Remove functionality
- Change response formats
- Become unavailable

Therefore adapters are important.

---

# 30. Provider Adapter Principle

Bad architecture:

    Business Logic
          ↓
    SerpApi-specific response

Better architecture:

    Business Logic
          ↓
    JobSource Interface
          ↓
    SerpApi Adapter
          ↓
    SerpApi

The same business logic should work if the provider changes.

---

# 31. Recommended Initial Source Flow

For V1:

    User
      ↓
    Search Request
      ↓
    Query Generator
      ↓
    Search Provider
      ↓
    Candidate URLs
      ↓
    URL Deduplication
      ↓
    Page Fetch
      ↓
    Extraction
      ↓
    Walk-in Detection
      ↓
    Normalization
      ↓
    Validation
      ↓
    Opportunity Deduplication
      ↓
    Freshness
      ↓
    Ranking
      ↓
    PostgreSQL

---

# 32. Source Selection Principles

When deciding whether to integrate a new source, evaluate:

## Coverage

Does it contain opportunities we cannot discover elsewhere?

## Quality

Is the information reliable?

## Freshness

How quickly do new opportunities appear?

## Structure

Does the source provide structured data?

## Accessibility

Can we access it appropriately?

## Cost

Does it require paid access?

## Rate limits

Can it support our expected volume?

## Stability

Is the API or page structure stable?

## Product value

Will integrating this source materially improve the user experience?

---

# 33. Source Evaluation Scorecard

A future source can be evaluated using:

    Coverage          25%
    Data Quality      20%
    Freshness         15%
    Accessibility     10%
    Structure         10%
    Reliability       10%
    Cost              5%
    Integration       5%

The exact weights can change.

The purpose is to make source decisions based on evidence instead of intuition.

---

# 34. V1 Source Strategy

The recommended V1 strategy is:

### Step 1

Start with one search-provider integration.

### Step 2

Build URL discovery.

### Step 3

Build page fetching.

### Step 4

Build generic extraction.

### Step 5

Build walk-in detection.

### Step 6

Measure results.

### Step 7

Identify which sources appear most frequently and produce the best opportunities.

### Step 8

Build dedicated adapters for the highest-value sources.

This is better than integrating dozens of sources before understanding actual product usage.

---

# 35. Important Research Conclusion

The research suggests that the best architecture is not:

    One API
       ↓
    Jobs Database

It should be:

    Multiple Discovery Sources
             ↓
       Source Adapters
             ↓
       Common Pipeline
             ↓
       Quality Layer
             ↓
       Opportunity Database

The quality layer is where the product creates long-term value.

---

# 36. Final Recommendation

For the real product, the recommended data-source strategy is:

    Search Providers
          +
    Direct Public Sources
          +
    Company Career Pages
          +
    Public ATS
          +
    Public Job APIs
          ↓
    Common Discovery Layer
          ↓
    Normalize
          ↓
    Validate
          ↓
    Deduplicate
          ↓
    Freshness
          ↓
    Rank

For V1, we should not build every integration immediately.

We should first prove:

    Search Query
       ↓
    Relevant Walk-in URLs
       ↓
    Correct Extraction
       ↓
    Correct Validation
       ↓
    Useful Results

After that, direct source integrations can be added based on measured coverage gaps.

---

# 37. Sources

## Google JobPosting

Google Search Central - Job Posting Structured Data

https://developers.google.com/search/docs/appearance/structured-data/job-posting

## Google Crawling / robots.txt

Google Crawling Documentation

https://developers.google.com/crawling/docs/robots-txt/robots-txt-spec

Google robots.txt creation and rules

https://developers.google.com/crawling/docs/robots-txt/create-robots-txt

## Adzuna

Adzuna API Overview

https://developer.adzuna.com/overview

Adzuna Search API

https://developer.adzuna.com/docs/search

Adzuna API Terms and Limits

https://developer.adzuna.com/docs/terms_of_service

## Greenhouse

Greenhouse Job Board API

https://docs.greenhouse.io/job-board.html

Greenhouse API Overview

https://support.greenhouse.io/hc/en-us/articles/10568627186203-Greenhouse-API-overview

## Lever

Lever Developer Documentation

https://hire.lever.co/developer/documentation

Lever Postings API

https://github.com/lever/postings-api

## Google Jobs Provider

SerpApi Google Jobs API

https://serpapi.com/google-jobs-api

SerpApi Google Jobs Listing API

https://serpapi.com/google-jobs-listing-api

---

# 38. Final Product Principle

The platform should remain source-independent.

The system should never be designed around:

    "We use API X."

It should be designed around:

    "We discover opportunities from multiple sources."

The source can change.

The discovery pipeline should remain stable.

The normalized opportunity model should remain stable.

The quality layer should remain the core product.

This allows V1 to focus on walk-ins while keeping the architecture ready for:

- Regular jobs
- Company-specific jobs
- Hiring drives
- Internships
- Campus drives
- Remote jobs
- Personalized job discovery
- Notifications
