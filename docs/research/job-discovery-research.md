# Job Discovery Research

## 1. Purpose

This document records the research done before building the Job Discovery Platform.

The purpose is to understand:

- Existing job and walk-in platforms
- How job opportunities can be discovered from the public internet
- Available job APIs and public job feeds
- Search-engine-based discovery
- Company career page discovery
- Data extraction challenges
- Duplicate opportunities
- Freshness and expired jobs
- Source transparency
- Trust and verification
- Technical and access limitations
- Where the product can be different

This research is for building a real product.

The goal is not to build a demo or a learning project.

---

# 2. Product Problem

The initial problem is:

> A job seeker wants to find relevant upcoming walk-in opportunities without manually searching many websites, job portals, company pages, and search results.

Walk-in opportunities are especially difficult because they are time-sensitive.

A useful walk-in listing normally needs information such as:

- Company
- Job title
- Location
- Interview date
- Interview time
- Venue
- Experience
- Skills
- Application information
- Original source

Finding a page is not enough.

The product must determine whether the opportunity is actually relevant, current, and useful.

---

# 3. Existing Product Research

## 3.1 WalkInInfo

WalkInInfo is an India-focused portal dedicated to walk-in interviews.

The website provides search by:

- Job title
- Company
- Keyword
- City

It emphasizes information such as:

- Venue
- Date
- Time
- Contact information

The platform also positions itself around verified employers and updated listings.

Source:

https://walkininfo.com/

Research observation:

The product is focused specifically on walk-ins rather than trying to cover every type of job.

This confirms that a focused walk-in product is a valid product category.

However, simply creating another walk-in listing website would not be enough differentiation.

Our product should focus more strongly on automated discovery, freshness, deduplication, and source transparency.

---

## 3.2 IndiaWalkins

IndiaWalkins is a broader Indian job portal with:

- Job search
- Location filters
- Categories
- Job types
- Company listings
- Candidate accounts
- Employer functionality

The site currently exposes jobs across many Indian locations and categories.

Source:

https://indiawalkins.in/

Research observation:

This is closer to a traditional job portal.

It demonstrates that a large amount of job information can be presented through filters, but it is not the same product direction as our discovery engine.

Our platform should not initially try to compete as a complete job portal.

---

## 3.3 Saarthi

Saarthi is another job-focused platform that includes opportunities such as walk-ins and hiring drives.

Its positioning includes reducing noise and focusing on useful opportunities.

Research observation:

Verification and relevance are important differentiators in the job-discovery space.

This supports our product principle:

    Relevance > Quantity

The platform should not try to win by showing the largest possible number of listings.

---

## 3.4 Broader Job Aggregators

There are many job aggregators that collect jobs from multiple sources.

The important product lesson is:

> Aggregation alone is not a strong enough differentiator.

If multiple websites contain the same job, showing the same opportunity multiple times creates noise.

Therefore our system needs a logical opportunity model that is independent of the original source.

Example:

    Company: Example Company
    Role: Java Backend Developer
    City: Bengaluru
    Event Date: 20 September

If the same opportunity is found on three websites, the user should ideally see one opportunity with multiple source references.

---

# 4. Market Insight

The existing market suggests several common approaches:

## Traditional job portal

Users search jobs that have been posted directly to the platform.

## Aggregator

The platform collects jobs from multiple sources.

## Curated job platform

The platform focuses on quality and manually or automatically verifies listings.

## Search assistant

The product helps users find opportunities across the wider web.

Our product should move toward the fourth model while retaining the trust characteristics of the third.

The long-term direction is:

    Job Discovery + Data Quality + Source Transparency

---

# 5. How Job Discovery Can Work

There is no single source containing every public job opportunity on the internet.

Google itself describes the web as having no central registry of all pages. Search engines discover pages through crawling, links, sitemaps, and other signals.

Source:

https://developers.google.com/search/docs/fundamentals/how-search-works

Therefore our system should not depend on one database.

A scalable discovery strategy should support multiple source types.

Possible source categories:

1. Search engines
2. Company career pages
3. ATS/job-board pages
4. Public job APIs
5. RSS/XML feeds
6. Public job aggregators
7. Other publicly accessible sources

---

# 6. Search Engine Discovery

Search engines are potentially one of the most useful discovery layers for V1.

Example user input:

    City: Bengaluru
    Technology: Java

The system can generate multiple queries:

    "walk-in interview" "Bengaluru" "Java"
    "walk-in drive" "Bengaluru" "Java"
    "walkin interview" "Bengaluru" "Java"
    "walk-in recruitment" "Bengaluru" "Java"
    "Java developer" "walk-in" "Bengaluru"
    "Java backend" "walk-in" "Bengaluru"

The search layer discovers candidate URLs.

The platform then fetches and processes the actual source pages.

This is important because the search engine should be treated as a discovery mechanism, not as the final source of truth.

The source of truth should remain the original job/event page whenever possible.

---

# 7. Google Search Research

Google explains that its search system:

1. Crawls pages
2. Indexes content
3. Serves relevant results

Google also states that there is no central registry of all web pages.

Source:

https://developers.google.com/search/docs/fundamentals/how-search-works

Important product implication:

We cannot assume that every job published on the internet will be discoverable immediately.

Some opportunities may:

- Not be indexed
- Be newly published
- Be blocked from crawling
- Require login
- Be removed
- Be poorly structured
- Exist only on a website that search engines do not surface well

Therefore search-engine discovery provides broad coverage but cannot guarantee complete coverage.

---

# 8. Google JobPosting Structured Data

Google supports a `JobPosting` structured-data format.

Employers and job sites can provide structured information about individual job postings.

Google recommends that JobPosting structured data be placed on the most specific page describing a single job.

Source:

https://developers.google.com/search/docs/appearance/structured-data/job-posting

This is useful for our extraction system.

When a source page contains valid JobPosting structured data, we may be able to extract structured information more reliably than parsing only visible text.

Potential fields include:

- Job title
- Description
- Hiring organization
- Location
- Employment information
- Date information

Important:

Structured data should be treated as a strong extraction signal, not automatically as proof that the opportunity is correct or active.

The actual page content and freshness still need validation.

---

# 9. Public Job APIs

Public job APIs can provide structured job data without requiring us to crawl every page ourselves.

One example is Adzuna.

Adzuna provides a REST API for job advertisements.

The API supports searching job listings using parameters such as:

- Keywords
- Location
- Categories
- Other search parameters

Sources:

https://developer.adzuna.com/overview

https://developer.adzuna.com/docs/search

Research observation:

An API-based source can be much easier to integrate than HTML extraction.

However, an API only gives us the jobs available through that provider.

It does not solve the complete public-web discovery problem.

Therefore APIs should be treated as one source in the discovery system rather than the entire discovery architecture.

---

# 10. Google Jobs Through Third-Party APIs

Services such as SerpApi provide an API for Google Jobs results.

Their Google Jobs API accepts search queries and location parameters and returns structured search results.

Source:

https://serpapi.com/google-jobs-api

Research observation:

This can potentially provide broad job discovery without directly implementing browser automation or scraping Google search-result pages ourselves.

However:

- It is a third-party dependency.
- It has usage limits and commercial considerations.
- It does not eliminate the need for source validation.
- It should not become the only discovery mechanism.

For V1, such a provider may be evaluated experimentally rather than hard-coded as the permanent architecture.

---

# 11. Company Career Pages

Company career pages are one of the highest-value sources because the company itself is publishing the opportunity.

A company career page can provide stronger source confidence than an unrelated aggregator.

Examples of common ATS-backed career systems include:

- Greenhouse
- Lever
- Other applicant tracking systems

Lever provides a public Postings API for published job postings and supports public job-site integrations.

Source:

https://hire.lever.co/developer/support

The Lever documentation specifically distinguishes its public Postings API from its authenticated account API.

Research implication:

Company/ATS sources should receive a high source-quality score when the opportunity can be traced directly to the employer or its official recruiting system.

---

# 12. Public ATS Feeds

Some companies publish jobs through ATS systems that expose structured public postings.

For example, Lever documents public postings and XML feeds for job boards.

Source:

https://hire.lever.co/developer/usecases

This suggests a useful future source strategy:

    Company
       ↓
    Official ATS
       ↓
    Public Job Posting
       ↓
    Our Discovery System

This can provide higher-quality structured data than generic web pages.

However, ATS sources are mainly useful for regular jobs.

Walk-in opportunities may still appear primarily on:

- Company pages
- Job portals
- Social posts
- Recruitment pages
- Aggregators
- Event pages

Therefore V1 should not depend only on ATS data.

---

# 13. Source Hierarchy

Not all sources should have equal trust.

A useful internal hierarchy is:

## Tier 1 — Official company source

Examples:

- Company career page
- Official recruitment page
- Official company hiring announcement

Highest source confidence.

## Tier 2 — Public ATS / structured employer source

Examples:

- Public Lever posting
- Public Greenhouse posting
- Other employer-controlled ATS pages

High source confidence.

## Tier 3 — Established job platform

Examples:

- Established job boards
- Established aggregators

Medium source confidence.

## Tier 4 — Other public websites

Examples:

- Blogs
- Recruitment websites
- Community pages

Lower source confidence unless corroborated.

This should influence ranking and trust signals.

It should not automatically mean that lower-tier sources are false.

---

# 14. Walk-in Discovery Is Different From Normal Job Discovery

Normal job postings can often remain active for days or weeks.

Walk-in opportunities are event-driven.

A walk-in opportunity normally has:

- Event date
- Interview time
- Venue
- Job role
- Eligibility
- Application/contact information

Therefore a walk-in system needs stronger freshness handling.

Example:

    Event Date: 20 September

After 20 September:

    Status = EXPIRED

The system should not continue presenting the opportunity as an upcoming walk-in.

---

# 15. Freshness Problem

Freshness is one of the most important product problems.

A page may still exist even after the event has happened.

Therefore:

    Page Exists ≠ Opportunity Is Active

The platform should maintain its own freshness information.

Important fields:

- discoveredAt
- firstSeenAt
- lastCheckedAt
- eventDate
- status

Possible states:

    ACTIVE
    AGING
    EXPIRED

Future versions may also detect:

    CANCELLED
    RESCHEDULED

---

# 16. Change Detection

An opportunity can change after discovery.

Example:

Initial page:

    Interview Date: 20 September
    Venue: Bengaluru

Later:

    Interview Date: 22 September
    Venue: Whitefield

The system should eventually detect these changes.

This is important because showing stale information can damage user trust.

Future change detection can compare:

- Previous extracted data
- Current extracted data
- Source page timestamp
- Structured-data timestamps
- Page content

---

# 17. Duplicate Problem

The same job can appear on many websites.

Example:

    Company: ABC
    Role: Java Developer
    City: Bengaluru
    Walk-in Date: 20 September

The same opportunity may appear on:

- Company website
- Job board A
- Job board B
- Recruitment website
- Search results

If we simply store every URL as a separate job, the user will see duplicates.

Therefore the platform needs two different concepts:

## Source

The webpage where the information was found.

## Opportunity

The logical real-world job/event.

One opportunity can have multiple sources.

Example:

    Opportunity
        |
        +-- Source A
        +-- Source B
        +-- Source C

This is a core architectural decision.

---

# 18. Deduplication Signals

Initial duplicate detection can use deterministic fields:

- Company
- Job title
- City
- Event date
- Venue
- Application URL

Additional signals:

- Description similarity
- Skills
- Experience
- Contact details

Future versions can use semantic similarity.

However, automatic merging should be conservative.

False merging is worse than showing a small number of duplicates.

---

# 19. Extraction Strategy

A single extraction method will not work for the entire web.

The extraction pipeline should use multiple strategies.

Recommended order:

    Structured Data
          ↓
    Known Source Parser
          ↓
    HTML / DOM Extraction
          ↓
    Text Pattern Extraction
          ↓
    AI-assisted Extraction
          ↓
    Validation

This approach avoids using expensive AI for every page.

---

# 20. Structured Data First

If a page contains useful structured data, use it.

Potential sources include:

- JobPosting JSON-LD
- OpenGraph metadata
- Standard HTML metadata
- RSS/XML feeds
- ATS APIs

Structured data is generally easier to parse than arbitrary page text.

However, structured data still needs validation.

---

# 21. Known Source Parsers

Some sources may be worth supporting with dedicated parsers.

Example:

    WalkInInfoParser
    CompanyCareerParser
    LeverParser
    GreenhouseParser

A known-source parser can provide higher extraction accuracy than generic parsing.

The architecture should allow source-specific extraction without coupling it to the entire system.

---

# 22. Generic Web Extraction

For unknown sources, the system should use a generic extraction pipeline.

Possible process:

    URL
      ↓
    Fetch HTML
      ↓
    Remove irrelevant elements
      ↓
    Extract visible text
      ↓
    Detect job/walk-in signals
      ↓
    Extract fields
      ↓
    Validate
      ↓
    Normalize

Irrelevant page elements may include:

- Navigation
- Advertisements
- Footer
- Cookie banners
- Unrelated links

---

# 23. Walk-in Detection

The word "walk-in" alone should not be enough.

Example:

> We are looking for candidates who can attend walk-in interviews in the future.

This may not represent a specific upcoming event.

Strong signals include combinations such as:

- Walk-in interview
- Walk-in drive
- Interview date
- Interview time
- Venue
- Recruitment drive
- Contact information

The more independent signals present, the stronger the classification.

---

# 24. Information Completeness

A useful opportunity should ideally contain:

    Company
    Role
    Location
    Date
    Source

Additional useful fields:

    Time
    Venue
    Experience
    Skills
    Application link
    Contact information

The system should not reject every opportunity with a missing optional field.

Instead, fields should have different importance.

Example:

Critical:

- Company
- Role
- Source
- Walk-in evidence

Important:

- Date
- Location

Useful:

- Venue
- Time
- Experience
- Skills

---

# 25. Validation

Validation should happen after extraction.

Examples:

## Date validation

If an event date is already in the past, the opportunity should not be treated as upcoming.

## Location validation

If the user searches Bengaluru and the extracted opportunity is clearly in Pune, it should not appear as a normal match.

## Role validation

A Java search should not return an unrelated sales role merely because the page contains the word "Java" somewhere.

## Walk-in validation

The page should contain sufficient evidence that it represents an actual walk-in opportunity.

---

# 26. Search Relevance

Search should not rely only on keyword matching.

Example:

User:

    Java Backend
    Bengaluru
    3-5 years

Good result:

    Java Backend Developer
    Bengaluru
    3-5 years

Potentially useful result:

    Java Microservices Engineer
    Bengaluru
    4 years

Poor result:

    Java Trainer
    Chennai
    1 year

Therefore ranking should consider multiple dimensions:

- Role
- Technology
- Location
- Experience
- Date
- Freshness
- Source quality
- Completeness

---

# 27. Trust Model

The product should separate two concepts.

## Extraction confidence

How confident are we that our system extracted the information correctly?

## Verification status

How much independent evidence do we have that the opportunity is genuine/current?

These are not the same thing.

A parser can extract a page perfectly while the original page itself is outdated.

Therefore:

    Extraction Confidence ≠ Verification

---

# 28. Possible Trust Signals

Instead of simply showing:

    VERIFIED

the product can show understandable evidence.

Examples:

    Official company source
    Recently checked
    Event date found
    Venue found
    Application link available
    Found on multiple sources

This gives users more information without making unsupported claims.

---

# 29. Source Transparency

Every opportunity should retain the original source.

The user should be able to answer:

> Where did this information come from?

The platform should show:

- Source name
- Original URL
- Last checked time

If multiple sources describe the same opportunity, the product may show:

    Found on 3 sources

This can become an important trust feature.

---

# 30. Crawling and Access Restrictions

The web cannot be treated as an unrestricted database.

Google documents that site owners can control crawling using mechanisms such as robots.txt and other access controls.

Sources:

https://developers.google.com/search/docs/crawling-indexing

https://developers.google.com/crawling/docs/robots-txt/intro

Important implication:

Our system should only use sources that we are permitted to access.

The platform should not:

- Bypass CAPTCHA
- Bypass login
- Circumvent access controls
- Ignore explicit access restrictions
- Use techniques intended to evade anti-bot protections

If a source cannot be accessed appropriately, the system should skip it or use another source.

---

# 31. Rate Limiting

Even when a page is publicly accessible, the platform should avoid aggressive crawling.

The system should:

- Use reasonable request rates
- Cache fetched pages where appropriate
- Avoid unnecessary repeated requests
- Use timeouts
- Handle server errors
- Back off when necessary

This protects both our system and source websites.

---

# 32. Search Provider vs Direct Crawling

There are two major discovery approaches.

## Approach A — Search provider first

    User Query
        ↓
    Search Provider
        ↓
    URLs
        ↓
    Fetch Source Pages
        ↓
    Extract
        ↓
    Validate

Advantages:

- Broad discovery
- Simple initial architecture
- Can discover unknown websites
- Good fit for V1

Disadvantages:

- Dependent on search provider
- Coverage is not guaranteed
- Results may change
- Provider costs/limits may apply

## Approach B — Crawl known sources directly

    Source List
        ↓
    Crawl
        ↓
    Extract

Advantages:

- More control
- Predictable sources
- Potentially better freshness
- Lower dependency on search engines

Disadvantages:

- Requires maintaining source-specific logic
- Cannot easily discover unknown websites
- More maintenance

---

# 33. Recommended Discovery Strategy

The research suggests a hybrid architecture.

V1 should start with:

    Search Discovery
          +
    Selected Direct Sources

Search discovery provides breadth.

Direct sources provide higher-quality and more predictable data.

Later:

    Search Sources
    Company Sources
    ATS Sources
    Public APIs
    RSS/XML
          ↓
    Common Discovery Pipeline

This avoids making the entire product dependent on one source.

---

# 34. Source Abstraction

The backend should use a common source interface.

Conceptually:

    JobSource
       |
       +-- SearchEngineSource
       +-- CompanyCareerSource
       +-- AtsSource
       +-- PublicApiSource
       +-- RssSource

Each source produces discovered opportunities/pages that go through the same downstream pipeline.

This allows new sources to be added without rewriting the complete system.

---

# 35. Search Provider Should Not Be the Product

A search API is an infrastructure dependency.

It should not become the product itself.

The product value should exist in the layers after discovery:

    Discovery
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
    Trust

This is where the platform can build its own intelligence and data quality.

---

# 36. Why We Should Not Depend Only on Adzuna

Adzuna provides a useful jobs API.

However, our initial product is specifically interested in walk-in discovery.

A general jobs API may not contain all walk-in events.

Therefore:

    Adzuna = Possible Future Source

not:

    Adzuna = Entire Discovery Engine

The same principle applies to other job APIs.

---

# 37. Why We Should Not Depend Only on Google Jobs

Google Jobs can provide broad job discovery through third-party services such as SerpApi.

However:

- It is not a complete source database.
- Search results can change.
- Walk-in-specific opportunities may not always appear.
- Third-party API costs and limits exist.

Therefore Google Jobs can be one discovery layer, not the complete platform.

---

# 38. Product Differentiation

Based on the research, the strongest differentiation opportunities are:

## 1. Relevance

Do not show everything.

Show opportunities that actually match the user's intent.

## 2. Freshness

Walk-in opportunities should be checked and expired automatically.

## 3. Deduplication

One real opportunity should not become five listings because five websites copied it.

## 4. Source Transparency

Always show where the information came from.

## 5. Trust Signals

Explain why an opportunity looks trustworthy.

## 6. Search Across Sources

The user should not need to search many websites manually.

## 7. Simple UX

The user should enter a few filters and get useful results.

---

# 39. What We Should NOT Build Initially

Research does not justify starting with a large infrastructure stack.

We should not initially build:

- Microservices
- Kafka
- Kubernetes
- Elasticsearch
- Complex AI pipelines
- Mobile applications
- Recruiter dashboards
- Payments
- Resume marketplace
- Automatic job applications

These can be introduced only when the product proves that they are needed.

---

# 40. Recommended V1 Architecture Direction

The research supports this initial architecture:

    User
      ↓
    Discovery API
      ↓
    Discovery Task
      ↓
    Query Generator
      ↓
    Search / Source Providers
      ↓
    Candidate URLs
      ↓
    Page Fetcher
      ↓
    Raw Documents
      ↓
    Extraction
      ↓
    Walk-in Detection
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
      ↓
    Search Results

This should initially be implemented as a modular monolith.

---

# 41. Recommended V1 Source Strategy

The first implementation should not attempt to support dozens of sources.

Start with a small number of reliable discovery mechanisms.

Recommended order:

### Phase 1

Search-provider-based discovery.

Goal:

    Query → URLs

### Phase 2

Generic page extraction.

Goal:

    URL → Clean text

### Phase 3

Walk-in detection.

Goal:

    Text → Walk-in candidate

### Phase 4

Structured extraction.

Goal:

    Candidate → JobPosting

### Phase 5

Validation and freshness.

Goal:

    JobPosting → Valid active opportunity

### Phase 6

Deduplication and ranking.

Goal:

    Many sources → Useful unique opportunities

### Phase 7

Direct source integrations.

Goal:

    Higher coverage + higher source quality

---

# 42. Research-Based Product Principle

The research supports the following principle:

> The product should not compete by collecting the most jobs. It should compete by helping users find the right opportunities with less noise and more confidence.

Therefore:

    Relevance > Quantity

    Freshness > Volume

    Trust > Hype

    Source Quality > Random Aggregation

    Product Quality > Infrastructure Complexity

---

# 43. Important Technical Decisions From Research

The following decisions are supported by the research:

### Decision 1

Use multiple source types.

Reason:

No single source contains every public job opportunity.

### Decision 2

Keep source and opportunity separate.

Reason:

The same real-world opportunity can appear on multiple websites.

### Decision 3

Keep raw and normalized data separate.

Reason:

Source information should remain traceable even after normalization.

### Decision 4

Treat search providers as discovery mechanisms.

Reason:

The original source page is more useful for validation and transparency.

### Decision 5

Use structured data when available.

Reason:

JobPosting and ATS data can provide more reliable structured information than arbitrary HTML.

### Decision 6

Validate extracted information.

Reason:

Extraction correctness and source correctness are different problems.

### Decision 7

Track freshness.

Reason:

Walk-in opportunities are event-driven and time-sensitive.

### Decision 8

Start with a modular monolith.

Reason:

The main uncertainty is product/data quality, not infrastructure scale.

---

# 44. Open Questions

These questions should be answered through implementation experiments rather than assumptions.

## Search provider

Which search provider gives sufficient coverage and acceptable cost for our use case?

## Walk-in coverage

How many useful walk-in opportunities can be discovered through search-based discovery?

## Extraction accuracy

How accurately can generic HTML extraction identify:

- Company
- Role
- Date
- Venue
- Experience
- Skills

## Duplicate accuracy

How accurately can we identify the same opportunity across multiple sources?

## Freshness

How often should different source types be rechecked?

## Source quality

Which sources consistently produce useful and current opportunities?

## Cost

What is the cost per successful discovery?

## Scale

How many searches and pages can the initial architecture process before additional infrastructure is required?

These should be measured after the first working discovery pipeline exists.

---

# 45. Research Conclusion

The research shows that building another static walk-in job listing website would not provide enough differentiation.

The stronger product opportunity is an automated discovery and intelligence layer.

The core system should:

    Discover
        ↓
    Extract
        ↓
    Understand
        ↓
    Validate
        ↓
    Deduplicate
        ↓
    Check Freshness
        ↓
    Rank
        ↓
    Show Source

V1 should remain focused on walk-in opportunities.

Once the discovery engine becomes reliable, the same architecture can expand to:

- Regular jobs
- Company-specific jobs
- Hiring drives
- Internships
- Campus hiring
- Personalized discovery
- Saved searches
- Notifications

The long-term product should become a job discovery platform rather than another job posting database.

---

# 46. Sources

## Existing Job / Walk-in Platforms

WalkInInfo  
https://walkininfo.com/

IndiaWalkins  
https://indiawalkins.in/

## Job APIs

Adzuna API  
https://developer.adzuna.com/overview

Adzuna Search API  
https://developer.adzuna.com/docs/search

## Google Search / Crawling

Google Search - How Search Works  
https://developers.google.com/search/docs/fundamentals/how-search-works

Google Crawling and Indexing  
https://developers.google.com/search/docs/crawling-indexing

Google Robots.txt Documentation  
https://developers.google.com/crawling/docs/robots-txt/intro

## Google JobPosting

Google Search Central - JobPosting Structured Data  
https://developers.google.com/search/docs/appearance/structured-data/job-posting

## Google Jobs API Provider

SerpApi Google Jobs API  
https://serpapi.com/google-jobs-api

## ATS / Public Job Sources

Lever Developer Documentation  
https://hire.lever.co/developer/documentation

Lever Public Postings / Job Board Documentation  
https://hire.lever.co/developer/usecases

---

# 47. Final Research Position

We should build the product around this architecture principle:

    Many Sources
         ↓
    One Discovery Pipeline
         ↓
    One Normalized Opportunity Model
         ↓
    One Trust / Freshness Layer
         ↓
    One Simple User Experience

The technology should support this product model.

The product should not be designed around a particular API, website, scraping technique, or infrastructure technology.

The source layer must be replaceable.

The opportunity model must remain stable.

The quality layer is the core product.

That is the foundation for future expansion from walk-in discovery into a broader job discovery platform.
