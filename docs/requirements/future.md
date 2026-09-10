# Future Requirements

This document describes the features that may be added after V1.

These features are not part of the initial implementation.

The priority is to make V1 reliable first and then expand the platform based on real user needs and data.

---

## 1. Product Direction

V1 focuses on one problem:

> Find relevant upcoming walk-in opportunities automatically.

The long-term product can become a broader job discovery platform that helps users discover opportunities from different sources without manually searching multiple websites.

The platform should remain focused on discovery rather than becoming a traditional job portal.

The long-term direction is:

    Discover → Understand → Filter → Verify → Apply

---

## 2. Regular Job Discovery

After walk-in discovery becomes reliable, the platform can support regular job openings.

Possible opportunity types:

- Regular jobs
- Walk-in interviews
- Hiring drives
- Off-campus drives
- Campus hiring
- Contract opportunities
- Internship opportunities

The existing opportunity model should allow these types to be added without redesigning the complete system.

Example:

    WALK_IN
    JOB
    HIRING_DRIVE
    INTERNSHIP
    CAMPUS_DRIVE

---

## 3. Company-Specific Search

Users may want to search for opportunities from a specific company.

Example:

> Show me all current Amazon opportunities.

The system should be able to discover opportunities related to a company across available public sources.

Possible search:

    Company: Amazon
    Role: Java Backend
    Location: Bengaluru

The result should combine relevant opportunities while avoiding duplicates.

---

## 4. Company Tracking

Users may be able to follow companies they are interested in.

Example:

> Follow Amazon

The platform could then show newly discovered opportunities from that company.

Possible future features:

- Follow company
- Company opportunity history
- Current openings
- Upcoming hiring drives
- Walk-in events
- Company-specific alerts

---

## 5. Saved Searches

Users may be able to save frequently used searches.

Example:

    Bengaluru + Java + 3-5 years

Another example:

    Gurgaon + Backend Developer + 2-5 years

Saved searches can later be used for automatic discovery and notifications.

---

## 6. Notifications

After saved searches are implemented, the platform can notify users when new relevant opportunities are discovered.

Possible channels:

- Email
- Push notification
- WhatsApp
- In-app notifications

Example:

> A new Java Backend walk-in opportunity was discovered in Bengaluru.

Notifications should be based on meaningful changes rather than sending every minor update.

---

## 7. Personalized Job Discovery

The platform may eventually understand a user's preferences.

Possible inputs:

- Preferred locations
- Preferred roles
- Skills
- Experience
- Preferred companies
- Job type
- Salary expectations

The system could then rank opportunities based on relevance.

The goal should be useful personalization, not excessive complexity.

---

## 8. Better Matching

Future versions may provide better matching between users and opportunities.

For example:

    User:
    Java
    Spring Boot
    Microservices
    4 years
    Bengaluru

    Opportunity:
    Java Backend Developer
    Spring Boot
    Microservices
    3-5 years
    Bengaluru

The platform could calculate a relevance score.

Possible factors:

- Skill match
- Role match
- Experience match
- Location match
- Company preference
- Job type
- Freshness

---

## 9. Salary and Compensation

For regular jobs, salary information can become an important search and ranking factor.

Possible fields:

- Minimum salary
- Maximum salary
- Salary currency
- Fixed compensation
- Variable compensation

Users could eventually filter by:

    Salary >= ₹15 LPA

The system should never invent salary information when it is not available from the source.

---

## 10. Location Intelligence

Future versions may provide better location handling.

Examples:

- Bengaluru
- Bangalore
- Gurgaon
- Gurugram
- Noida
- Delhi NCR

The platform could normalize location names internally.

It may also support:

- Nearby locations
- Distance-based search
- Remote jobs
- Hybrid jobs
- Office locations

---

## 11. Source Expansion

The discovery engine can support additional public sources over time.

Potential sources:

- Company career pages
- Public job boards
- Public job aggregators
- Search engines
- RSS feeds
- Public APIs
- Employer websites

Each source should be implemented behind a common source abstraction.

The core discovery pipeline should not depend on one particular website.

---

## 12. Source Quality Scoring

As the number of sources increases, the platform may learn which sources consistently provide useful information.

Possible source-quality factors:

- Freshness
- Accuracy
- Completeness
- Availability
- Duplicate rate
- Historical reliability

This can be used as one factor in ranking.

A source should not automatically be considered trustworthy simply because it is popular.

---

## 13. Advanced Deduplication

Duplicate detection can become more sophisticated as the platform grows.

Future techniques may compare:

- Company
- Job title
- Location
- Event date
- Venue
- Skills
- Experience
- Description
- Application URL
- Source content

Semantic similarity may eventually be used when simple matching is not enough.

The system should still avoid merging two genuinely different opportunities.

---

## 14. Change Detection

A future version may track changes on source pages.

For example:

    Before:
    Walk-in Date: 20 September

    After:
    Walk-in Date: 22 September

The platform could detect the change and update the opportunity.

Other changes may include:

- Venue changed
- Interview time changed
- Experience requirement changed
- Application link changed
- Event cancelled

Important changes could eventually trigger notifications.

---

## 15. Opportunity History

The platform may maintain an internal history of important opportunity changes.

Example:

    Opportunity Created
          ↓
    Event Date Updated
          ↓
    Venue Updated
          ↓
    Event Completed

This can help with:

- Debugging
- Trust
- Change detection
- Analytics
- Source reliability

---

## 16. Verification System

A more advanced verification system may be introduced later.

Possible verification signals:

- Official company source
- Multiple independent sources
- Matching event information
- Recently checked source
- Valid application page
- Consistent company information

Possible future states:

    UNVERIFIED
    PARTIALLY_VERIFIED
    VERIFIED

The meaning of each state should be clearly defined before being exposed to users.

The platform should never use verification labels only for marketing purposes.

---

## 17. Application Tracking

The platform may eventually allow users to track opportunities they are interested in.

Possible states:

    SAVED
    INTERESTED
    APPLIED
    INTERVIEW_SCHEDULED
    REJECTED
    SELECTED
    CLOSED

This would help users manage their job search without turning the platform into a full recruitment system.

---

## 18. Resume-Based Matching

Users may eventually provide their resume.

The system could extract:

- Skills
- Experience
- Roles
- Technologies
- Education
- Certifications

The platform could then suggest relevant opportunities.

Example:

> Based on your profile, these Java Backend opportunities are highly relevant.

Resume processing should only be introduced when there is a clear product benefit.

---

## 19. AI-Assisted Extraction

AI may be introduced for difficult extraction cases.

Possible uses:

- Extracting structured information from messy pages
- Understanding job descriptions
- Identifying skills
- Detecting experience requirements
- Detecting walk-in details
- Resolving ambiguous information

AI should not automatically replace deterministic parsing where simple rules are sufficient.

A possible future pipeline:

    HTML
      ↓
    Text Extraction
      ↓
    Rule-Based Extraction
      ↓
    AI Extraction for Difficult Cases
      ↓
    Validation
      ↓
    Normalization

AI output must still go through validation.

---

## 20. AI-Assisted Matching

AI may eventually help understand semantic relationships.

For example:

    "Java Backend Developer"

may be matched with:

    "Software Engineer - Java Platform"

even when the exact words are different.

Similarly:

    Spring Boot
    Spring Framework
    Java Microservices

may be understood as related technologies.

This should improve discovery quality without hiding the actual source information.

---

## 21. Search Improvements

Future search capabilities may include:

- Natural language search
- Multiple locations
- Multiple technologies
- Company filters
- Salary filters
- Remote/hybrid filters
- Job type filters
- Date filters
- Experience ranges
- Skill-based search

Example:

> Find Java backend jobs in Bengaluru for 3-5 years experience posted in the last 7 days.

The system should convert this into structured search criteria.

---

## 22. User Accounts

User accounts may be added when personalized features become necessary.

Possible features:

- Login
- Profile
- Preferences
- Saved searches
- Followed companies
- Saved opportunities
- Application tracking
- Notification settings

User accounts are not required for the initial discovery engine.

---

## 23. Analytics

Future analytics can help understand whether the discovery engine is actually useful.

Possible metrics:

- Search count
- Discovery success rate
- Relevant opportunity rate
- Duplicate rate
- Expired opportunity rate
- Source quality
- Click-through rate
- Save rate
- Application click rate

Analytics should help improve the product rather than simply increase the number of jobs displayed.

---

## 24. Search and Recommendation Infrastructure

As the number of opportunities grows, PostgreSQL may no longer be sufficient for every search requirement.

Future infrastructure may include:

- Elasticsearch
- OpenSearch
- Redis
- Kafka
- Background workers
- Distributed processing

These should only be introduced when the actual scale or product requirements justify them.

The initial system should remain simple.

---

## 25. Scalability

If discovery volume becomes large, the system may evolve from a modular monolith into multiple services.

Possible future services:

    Discovery Service
    Source Service
    Fetching Service
    Extraction Service
    Normalization Service
    Validation Service
    Deduplication Service
    Ranking Service
    Search Service
    Notification Service

This should happen only when there is a real reason to separate services.

Microservices are not a requirement for the initial product.

---

## 26. Infrastructure Evolution

The infrastructure may eventually use:

- Docker
- Kubernetes
- AWS
- Managed PostgreSQL
- Redis
- Kafka
- Object storage
- Search infrastructure
- CI/CD pipelines
- Monitoring
- Centralized logging

The technology choice should follow actual system requirements.

The product should not introduce infrastructure complexity simply because it is available.

---

## 27. International Expansion

The initial product may focus on India.

Later it could support other countries and regions.

Possible future requirements:

- Country-specific sources
- Local location normalization
- Local date/time formats
- Currency handling
- Country-specific job sources
- Regional job terminology

The core discovery architecture should remain source-independent enough to support this expansion.

---

## 28. Mobile Application

A mobile application may be considered after the web product proves useful.

Possible platforms:

- Android
- iOS

The mobile application would consume the same backend APIs.

The backend should remain independent of the frontend platform.

---

## 29. Product Expansion Principle

Future features should follow a simple rule:

> Solve the core discovery problem first, then expand based on real user needs.

The platform should not become overloaded with features before the discovery engine is reliable.

Priority should generally follow:

    Discovery Quality
        ↓
    Freshness
        ↓
    Relevance
        ↓
    Trust
        ↓
    Personalization
        ↓
    Notifications
        ↓
    Application Tracking
        ↓
    Advanced AI
        ↓
    Large-Scale Infrastructure

---

## 30. Long-Term Vision

The long-term vision is to build a job discovery and intelligence platform rather than another traditional job portal.

The platform should help users answer questions such as:

> What relevant opportunities are available for me right now?

> Which companies are hiring for my skills?

> Are there upcoming walk-in drives near my preferred location?

> Is this opportunity still active?

> Where did this opportunity originally come from?

> How trustworthy and fresh is this information?

The product should make job discovery easier by reducing the need to manually search across many different sources.

The core principle remains:

    Discover better opportunities.
    Reduce noise.
    Keep information fresh.
    Show the source.
    Help users make better job-search decisions.
