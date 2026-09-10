# Competitor Research

## 1. Purpose

This document analyzes existing products that are relevant to the Job Discovery Platform.

The goal is not to copy existing products.

The goal is to understand:

* What already exists
* How competitors discover jobs
* What users can currently do
* What competitors do well
* Where their approach is limited
* Where there is room for differentiation
* What we should and should not build

The product is intended to become a real job discovery platform, so competitor research should directly influence product decisions.

---

# 2. Competitive Landscape

The current market can broadly be divided into four categories:

1. Traditional job portals
2. Walk-in focused platforms
3. Job aggregators
4. AI-powered job discovery/application platforms

Our initial product sits closest to categories 2 and 3, but the long-term direction is closer to a job discovery and intelligence layer.

---

# 3. WalkInInfo

## Website

WalkInInfo

## Product Type

Walk-in interview focused job platform.

## What It Does

WalkInInfo is specifically focused on walk-in interviews in India.

Users can search using:

* Job title
* Company
* Keyword
* City

The platform emphasizes information such as:

* Venue
* Date
* Time
* Contact information

It positions itself as a focused walk-in portal rather than a general job board.

## Strengths

### 1. Narrow focus

The platform focuses specifically on walk-in opportunities.

This is useful because users do not have to search through unrelated regular jobs.

### 2. Simple search

The search experience is straightforward.

Users can search by:

* Keyword
* City

### 3. Event information

The platform emphasizes the practical information a candidate needs before attending:

* Date
* Time
* Venue

### 4. Freshness positioning

The platform communicates that walk-ins are updated regularly.

## Weaknesses / Opportunity for Us

The product is primarily presented as a listing portal.

Our opportunity is to build a deeper discovery layer behind the listing.

Instead of only maintaining a set of listings, our system should automatically discover opportunities from multiple public sources.

The product can then:

* Discover
* Extract
* Normalize
* Validate
* Deduplicate
* Check freshness
* Rank

## What We Learn

A dedicated walk-in experience is valuable.

However, the real differentiation should come from the quality of the discovery engine rather than simply having a walk-in category.

---

# 4. Saarthi

## Website

Saarthi

## Product Type

Job platform with a strong focus on fresher opportunities and walk-in drives.

## What It Does

Saarthi maintains walk-in interview listings across Indian cities.

The platform currently exposes information such as:

* Company
* Role
* Location
* Walk-in date
* Date ranges
* Salary where available

It also provides city-oriented discovery.

## Strengths

### 1. Large walk-in focus

Saarthi has dedicated walk-in pages and covers multiple cities.

### 2. Practical information

Listings contain information that helps candidates decide whether to attend.

### 3. Fresher orientation

The product strongly targets freshers and early-career candidates.

### 4. Content around job search

The platform combines listings with informational content and job-search guidance.

## Weaknesses / Opportunity for Us

The platform is still fundamentally a listing/aggregation experience.

Our product can focus more heavily on:

* Automated discovery
* Source tracking
* Opportunity-level deduplication
* Freshness
* Source quality
* Evidence-based trust

## What We Learn

Walk-in discovery should be treated as an event-discovery problem, not just a normal job-listing problem.

---

# 5. Hyriko

## Product Type

Job and internship aggregator focused on Indian students and freshers.

## What It Does

Hyriko states that it collects job openings directly from company career pages and public ATS sources.

It currently aggregates openings from hundreds of companies and presents them in one searchable feed.

The platform supports filters such as:

* Role
* Company
* Skill
* Location
* Job type
* Work mode
* Pay

It also states that it deduplicates cross-posted roles and removes spam.

## Strengths

### 1. Direct company sources

This is one of the strongest aspects of the product.

Instead of depending only on traditional job boards, Hyriko says it pulls roles from company career pages and public ATS feeds.

### 2. Aggregation

Users do not have to manually check many company career pages.

### 3. Deduplication

The platform explicitly addresses cross-posted roles.

This validates deduplication as an important product feature.

### 4. Freshness

The platform presents jobs as live/current openings and updates them regularly.

### 5. Direct application

Listings can link candidates directly to the employer's application page.

## Weaknesses / Opportunity for Us

Hyriko focuses heavily on:

* Students
* Freshers
* Internships
* Regular jobs

Our initial product is different.

We want to solve:

> Find upcoming walk-in opportunities automatically.

Walk-ins require additional event-specific logic:

* Interview date
* Interview time
* Venue
* Event status
* Event expiry
* Rescheduling

## What We Learn

Hyriko validates an important architecture decision:

> Source aggregation + normalization + deduplication + freshness can create value beyond a traditional job board.

We should use the same general principle but apply it initially to walk-in opportunities.

---

# 6. Hopin

## Product Type

AI-powered job discovery and application platform.

## What It Does

Hopin positions itself as an AI job-search assistant for Indian students, freshers and early-career candidates.

Its product includes:

* Job discovery
* Authenticity checking
* Resume tailoring
* Application submission
* Application tracking
* Follow-up
* WhatsApp interaction

It also states that it crawls job openings and looks for opportunities that may not appear on traditional job boards.

## Strengths

### 1. Discovery beyond traditional job boards

This is highly relevant to our product.

Hopin's positioning shows that there is product value in discovering opportunities beyond the obvious job-board listings.

### 2. Authenticity

The platform emphasizes checking opportunities before applying.

This reinforces the importance of trust.

### 3. End-to-end workflow

Hopin goes beyond discovery into:

* Resume customization
* Application
* Follow-up
* Tracking

## Weaknesses / Opportunity for Us

The product is broad and application-oriented.

Our initial product should remain narrower.

We do not need to solve:

* Resume generation
* Auto-apply
* Recruiter outreach
* Application automation

The first problem we need to solve is discovery quality.

## What We Learn

A discovery engine can eventually become the foundation for a much larger job-search product.

However:

> Discovery quality should be proven before building application automation.

---

# 7. IndiaWalkins

## Product Type

General Indian job portal with walk-in/job-oriented positioning.

## What It Does

The platform provides:

* Job search
* Job listings
* Company information
* Job categories
* Employer posting

It contains jobs across multiple locations and job categories.

## Strengths

### 1. Broad coverage

It is not restricted to one role or technology.

### 2. Employer functionality

Companies can post jobs.

### 3. Traditional job portal features

The product supports the standard job-board model.

## Weaknesses / Opportunity for Us

The broader job-board model creates a different product experience from what we want.

Our initial product should not require:

* Recruiter accounts
* Employer dashboards
* Manual job posting
* Large employer workflows

Instead, we want automated public-web discovery.

## What We Learn

We should avoid becoming a traditional job portal too early.

---

# 8. WorkIndia

## Product Type

Large Indian employment platform focused heavily on frontline, blue-collar and service-sector hiring.

## Strengths

* Large candidate base
* Direct hiring orientation
* Employer functionality
* Job discovery
* Candidate communication

## Difference From Our Product

WorkIndia is a broad employment marketplace.

Our initial product is not intended to compete with it directly.

Our focus is:

```
Public Web Discovery
        +
Relevance
        +
Freshness
        +
Source Transparency
```

## What We Learn

A large marketplace requires substantial supply-side and demand-side infrastructure.

We should not attempt this model in V1.

---

# 9. LinkedIn

## Product Type

Professional network with job discovery.

## Strengths

* Large professional user base
* Strong company presence
* Job search
* Recruiter ecosystem
* Professional profiles
* Networking
* Personalized recommendations

## Weaknesses / Opportunity for Us

LinkedIn is a closed ecosystem compared with the public web.

Our product can focus on discovering opportunities across public sources rather than requiring the user to search inside one platform.

We should not attempt to replace LinkedIn's professional network.

Instead, our product should solve a different problem:

> Find relevant opportunities across multiple public sources.

---

# 10. Naukri

## Product Type

Large Indian job portal.

## Strengths

* Very large job database
* Strong recruiter ecosystem
* Search and filters
* Candidate profiles
* Recruiter access
* Established brand

## Difference From Our Product

Naukri is fundamentally a job marketplace.

Our initial product should be a discovery engine.

The user should not need to maintain a large profile simply to discover a public opportunity.

## What We Learn

We should not compete on:

```
"We have more jobs."
```

We should compete on:

```
"We help you find the right opportunities with less noise."
```

---

# 11. Competitor Comparison

| Product        | Main Focus             | Walk-ins | Public-Web Discovery |      Company Sources |        Deduplication |          Freshness | Application Workflow |
| -------------- | ---------------------- | -------: | -------------------: | -------------------: | -------------------: | -----------------: | -------------------: |
| WalkInInfo     | Walk-in jobs           |   Strong |      Limited/unclear | Not core positioning | Not core positioning | Strong positioning |                Basic |
| Saarthi        | Jobs + walk-ins        |   Strong |          Aggregation | Not core positioning | Not core positioning |    Regular updates |                Basic |
| Hyriko         | Jobs + internships     |  Limited |               Strong |               Strong |                  Yes |             Strong |         Direct apply |
| Hopin          | AI job assistant       | Not core |               Strong |        Strong/varied |  Not primary message |             Strong |          Very strong |
| IndiaWalkins   | Job portal             |     Some |      Limited/unclear |    Employer listings |          Not primary |   Regular listings |                Basic |
| WorkIndia      | Employment marketplace |     Some |             Not core |      Employer-driven |       Platform-based |             Strong |               Strong |
| LinkedIn       | Professional network   |     Some |     Closed ecosystem |               Strong |       Platform-based |             Strong |               Strong |
| Naukri         | Job marketplace        |     Some |     Closed ecosystem |               Strong |       Platform-based |             Strong |               Strong |
| Our Product V1 | Walk-in discovery      |   Strong |               Strong |               Future |               Strong |             Strong |      Source redirect |

---

# 12. Competitor Gap

The research shows that different products solve different parts of the job-search problem.

Some are strong at:

* Walk-ins
* Job aggregation
* Company career aggregation
* AI application
* Recruiter marketplaces

But our product can combine a different set of capabilities:

```
Public Web Discovery
        +
Walk-in Detection
        +
Event Extraction
        +
Deduplication
        +
Freshness
        +
Source Transparency
        +
Relevance Ranking
```

This combination is the core opportunity.

---

# 13. Our Initial Differentiation

The product should not position itself as:

> Another job portal.

Instead:

> A discovery engine for finding relevant job opportunities across the public web.

For V1:

> Find upcoming walk-in opportunities without manually searching multiple websites.

---

# 14. Differentiation Pillar 1 — Discovery

Competitors may already have their own job databases.

Our system should actively discover opportunities from multiple public sources.

Conceptually:

```
User Search
    ↓
Search Sources
    ↓
Candidate URLs
    ↓
Opportunity Extraction
```

This allows the product to find opportunities that are not already stored in our own manually maintained database.

---

# 15. Differentiation Pillar 2 — One Opportunity, Multiple Sources

A real-world opportunity should be represented as one logical entity.

Example:

```
ABC Company
Java Developer
Bengaluru
Walk-in
20 September
```

If found on:

* Company website
* Job board
* Recruitment website

the user should ideally see one opportunity.

Internally:

```
Opportunity
   ├── Source A
   ├── Source B
   └── Source C
```

This reduces noise.

---

# 16. Differentiation Pillar 3 — Freshness

A walk-in opportunity is useful only while it is still relevant.

The system should track:

* First discovered
* Last checked
* Event date
* Current status

Possible states:

```
ACTIVE
AGING
EXPIRED
```

The product should automatically stop treating past events as upcoming opportunities.

---

# 17. Differentiation Pillar 4 — Source Transparency

Every opportunity should answer:

> Where did this information come from?

The result should provide:

* Source name
* Original URL
* Last checked time

If multiple sources are available:

```
Found on 3 sources
```

This provides context to the user.

---

# 18. Differentiation Pillar 5 — Trust

We should avoid unsupported claims such as:

```
100% Verified
Guaranteed Job
Official
```

Instead, show evidence.

For example:

```
Official company source
Recently checked
Event date found
Venue available
Multiple sources found
```

Trust should be evidence-based.

---

# 19. Differentiation Pillar 6 — Relevance

Showing 1,000 jobs is not necessarily better than showing 20 useful jobs.

Ranking should consider:

* Role match
* Technology match
* Location
* Experience
* Event date
* Freshness
* Source quality
* Completeness

The product should optimize for:

```
Useful Results > Maximum Results
```

---

# 20. What We Should Copy

We should learn from competitors, not copy their products.

Useful ideas worth adopting:

### From WalkInInfo

* Dedicated walk-in experience
* Event-specific information
* Simple search

### From Saarthi

* City-based walk-in discovery
* Fresher-friendly presentation
* Walk-in-focused content

### From Hyriko

* Company career-page aggregation
* Public ATS sources
* Deduplication
* Direct employer links
* Freshness

### From Hopin

* Discovery beyond traditional job boards
* Authenticity/trust checks
* Long-term potential for personalized job discovery

### From large job portals

* Strong filtering
* Search relevance
* Company discovery
* User-friendly browsing

---

# 21. What We Should Avoid

We should not initially copy:

* Large recruiter marketplace
* Employer dashboard
* Resume marketplace
* Auto-apply
* Complex candidate profiles
* Premium subscriptions
* Huge infrastructure
* Large-scale social networking
* Too many job categories

These features increase complexity without solving the initial problem.

---

# 22. Product Positioning

The initial product should be positioned as:

> A smart job discovery platform that automatically finds relevant opportunities from across the public web.

For V1:

> Automatically discover upcoming walk-in interviews matching your city, role, and experience.

The product should feel like a search engine for job opportunities rather than another static job board.

---

# 23. Strategic Advantage

The long-term advantage should not be the frontend.

It should be the data pipeline.

The important system is:

```
Sources
   ↓
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
Search
```

If this pipeline becomes reliable, the same infrastructure can later support:

* Walk-ins
* Regular jobs
* Hiring drives
* Internships
* Company-specific discovery
* Campus drives
* Remote jobs

---

# 24. Competitor Risk

The biggest risk is not that another company has a similar UI.

The bigger risk is that existing aggregators improve their discovery and data-quality systems.

Therefore our differentiation should be based on product quality rather than a temporary feature.

Important areas to build deeply:

* Better discovery queries
* Better extraction
* Better deduplication
* Better freshness
* Better ranking
* Better source transparency

---

# 25. Final Competitive Conclusion

The market already has:

* Walk-in portals
* Large job boards
* Job aggregators
* Company-job aggregators
* AI job assistants

Therefore simply building:

```
"A website containing walk-in jobs"
```

is not enough.

Our product should instead build:

```
Public Web Discovery
        ↓
Structured Opportunities
        ↓
Quality Control
        ↓
Deduplication
        ↓
Freshness
        ↓
Trust
        ↓
Simple Search Experience
```

The strongest initial positioning is:

> Find the right job opportunities without searching everywhere yourself.

For V1, this means:

> Find upcoming walk-in opportunities automatically, remove duplicate/noisy results, keep them fresh, and always show the original source.

Once this engine becomes reliable, it becomes the foundation for the broader job discovery platform.

