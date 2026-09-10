# Database Architecture

## Purpose

PostgreSQL is the primary source of truth for the Job Discovery Platform.

The database stores:

- Discovered job opportunities
- Companies
- Job sources
- Source URLs
- Raw discovery information
- Discovery tasks
- Skills
- Opportunity freshness
- Relationships between opportunities and sources

The database should support the V1 discovery pipeline without introducing unnecessary complexity.

The initial database should be designed so that it can evolve later as the product expands to regular jobs, company-specific searches, saved searches, notifications, and personalized recommendations.

---

## 1. Database Choice

V1 uses:

- PostgreSQL
- Spring Data JPA
- Hibernate
- Maven database migrations

PostgreSQL is preferred because the platform needs:

- Relational relationships
- Strong consistency
- Transactions
- Indexing
- Flexible querying
- Constraints
- Good support for structured job data
- JSON support where useful

PostgreSQL is the source of truth.

Redis and a dedicated search engine are not required initially.

---

## 2. Core Database Model

The initial domain can be represented using these main entities:

- Company
- JobOpportunity
- JobSource
- JobOpportunitySource
- RawDocument
- DiscoveryTask
- Skill
- JobOpportunitySkill

Conceptually:

Company
→ JobOpportunity
→ JobOpportunitySource
→ JobSource
→ RawDocument

JobOpportunity
→ JobOpportunitySkill
→ Skill

DiscoveryTask
→ discovered/processed JobOpportunity records

---

## 3. Company

The `company` table stores normalized company information.

Possible fields:

- id
- name
- normalized_name
- website
- created_at
- updated_at

Example:

name:
ABC Technologies Pvt Ltd

normalized_name:
abc technologies

The normalized name helps identify the same company across different sources.

The original company name should still be preserved where useful.

---

## 4. Job Opportunity

The `job_opportunity` table represents the normalized job opportunity shown to users.

Possible fields:

- id
- company_id
- title
- normalized_title
- description
- city
- normalized_city
- country
- job_type
- minimum_experience
- maximum_experience
- event_date
- event_start_time
- event_end_time
- venue
- application_url
- application_method
- walk_in_status
- freshness_status
- extraction_confidence
- verification_status
- first_discovered_at
- last_seen_at
- expires_at
- created_at
- updated_at

The exact schema can evolve during implementation.

The important principle is that this table contains normalized product-level information rather than raw webpage content.

---

## 5. Opportunity Type

The system should not permanently assume that every future opportunity is a walk-in.

V1 primarily supports:

WALK_IN

The model should leave room for future types such as:

- REGULAR_JOB
- HIRING_DRIVE
- INTERNSHIP
- CAMPUS_DRIVE
- OTHER

This allows the platform to expand without redesigning the complete domain model.

---

## 6. Walk-in Status

A walk-in classification can be represented as:

- WALK_IN
- NOT_WALK_IN
- UNCERTAIN

Only appropriate opportunities should normally reach the user-facing V1 result set.

The raw evidence supporting the classification should remain traceable through the source/raw document information.

---

## 7. Freshness Status

Freshness should be stored explicitly.

Possible values:

- ACTIVE
- AGING
- EXPIRED

The application should not rely only on the UI to determine whether a job is expired.

Freshness should be part of the backend domain model.

Example:

event_date:
2026-09-15

freshness_status:
ACTIVE

After the event has passed:

freshness_status:
EXPIRED

---

## 8. Verification Status

Extraction confidence and verification status should remain separate.

Possible verification values:

- UNKNOWN
- SOURCE_CONFIRMED
- VERIFIED

The exact verification process can become more sophisticated later.

For V1, the platform should avoid presenting an opportunity as officially verified unless there is a meaningful basis for doing so.

---

## 9. Job Source

The `job_source` table represents the source from which an opportunity was discovered.

Examples of source types:

- SEARCH_ENGINE
- COMPANY_CAREER_PAGE
- ATS
- JOB_BOARD
- AGGREGATOR
- PUBLIC_API
- OTHER

Possible fields:

- id
- name
- source_type
- base_url
- quality_score
- active
- created_at
- updated_at

Source-level quality can later be calculated from historical performance.

---

## 10. Job Opportunity Source

One opportunity may appear on multiple websites.

Therefore, the relationship between opportunities and sources should be many-to-many.

The `job_opportunity_source` table connects them.

Possible fields:

- id
- opportunity_id
- source_id
- source_url
- normalized_url
- is_primary
- discovered_at
- last_seen_at
- created_at
- updated_at

Example:

One Java walk-in opportunity:

Opportunity
→ Company career page
→ LinkedIn
→ Job board
→ Recruitment website

These should not necessarily become four separate opportunities.

They can become one opportunity with multiple source references.

---

## 11. Primary Source

An opportunity may have multiple sources, but one source can be marked as primary.

For example:

Official company career page
→ primary source

Job aggregator
→ secondary source

The primary source can be preferred when showing the source to users.

However, secondary sources should not automatically be discarded.

They can provide:

- additional confirmation
- alternative application information
- discovery coverage
- change detection signals

---

## 12. Raw Document

The `raw_document` model stores information about fetched source content.

Possible fields:

- id
- source_id
- source_url
- normalized_url
- content_hash
- content_type
- http_status
- raw_content
- discovered_at
- fetched_at
- created_at

The raw document is different from the normalized opportunity.

Example:

Raw document:

HTML page from a recruitment website.

Normalized opportunity:

Java Backend Developer
Bengaluru
3-5 years
Walk-in on a specific date

The raw information allows future extraction logic to be improved without losing the original source data.

---

## 13. Raw Data Storage Strategy

V1 can store manageable raw content in PostgreSQL if appropriate.

However, raw webpages can become large.

As the system scales, raw documents can move to object storage.

Future architecture:

PostgreSQL
→ Metadata

Object Storage
→ Large raw documents

The database should therefore avoid becoming dependent on storing very large blobs everywhere.

This migration can happen later.

---

## 14. Content Hash

A content hash can be stored for fetched documents.

For example:

content_hash

This helps detect whether the content has changed.

If the same URL is fetched again:

Same hash
→ probably unchanged

Different hash
→ page changed

This becomes useful for future change detection.

---

## 15. Discovery Task

The `discovery_task` table represents one backend discovery operation.

Possible fields:

- id
- status
- search parameters
- queries generated
- urls discovered
- urls fetched
- urls failed
- opportunities extracted
- opportunities rejected
- duplicates removed
- started_at
- completed_at
- created_at
- updated_at

The exact metrics can be normalized into separate tables later if required.

V1 should keep task tracking simple.

---

## 16. Discovery Task Status

Possible states:

- CREATED
- RUNNING
- COMPLETED
- PARTIALLY_COMPLETED
- FAILED

A task can be `PARTIALLY_COMPLETED` when some sources or pages fail but useful opportunities are still discovered.

This is preferable to treating every individual source failure as a complete task failure.

---

## 17. Search Parameters

The discovery task should retain the parameters used for discovery.

For example:

city:
Bengaluru

role:
Java Backend Developer

skills:
Java, Spring Boot

minimum experience:
2

maximum experience:
5

date range:
Next 30 days

This provides traceability.

It also helps debug why a particular opportunity was discovered.

Search parameters can initially be stored as JSON if that keeps the schema simple.

Frequently queried domain fields should remain normal columns in the appropriate domain tables.

---

## 18. Skill

The `skill` table stores normalized technology or skill names.

Examples:

- Java
- Spring Boot
- Kafka
- Microservices
- SQL
- AWS
- Docker

Possible fields:

- id
- name
- normalized_name
- created_at

Skills should be normalized so that:

Spring Boot

and

springboot

do not unnecessarily become separate skills.

---

## 19. Job Opportunity Skill

The relationship between jobs and skills is many-to-many.

The `job_opportunity_skill` table connects:

JobOpportunity
→ Skill

Possible fields:

- opportunity_id
- skill_id
- required
- created_at

This makes technology-based filtering easier.

For example:

Find active walk-in opportunities requiring:

Java

Spring Boot

Kafka

---

## 20. Relationships

The main relationships are:

Company
→ has many Job Opportunities

Job Opportunity
→ belongs to one Company

Job Opportunity
→ has many Job Opportunity Sources

Job Source
→ has many Job Opportunity Sources

Job Opportunity
→ has many Skills

Skill
→ belongs to many Job Opportunities

Job Source
→ has many Raw Documents

Discovery Task
→ discovers/processes many candidate URLs and opportunities

The exact foreign-key design can evolve during implementation.

---

## 21. Database Constraints

The database should enforce important invariants wherever possible.

Examples:

- company name should not be empty
- opportunity title should not be empty
- source URL should not be empty
- foreign keys should be valid
- required enum/status values should be valid
- duplicate source URLs should be controlled
- duplicate opportunity fingerprints should be controlled where practical

Application validation and database constraints should complement each other.

---

## 22. Idempotency

The discovery pipeline may process the same page multiple times.

The database should therefore support idempotent writes.

Useful identifiers include:

- normalized source URL
- content hash
- company
- normalized title
- city
- event date
- venue
- application URL

A unique constraint can be used where the business rule is strong enough.

For fuzzy opportunity matching, application-level deduplication should be used rather than relying only on database uniqueness.

---

## 23. Opportunity Fingerprint

A normalized opportunity fingerprint can help identify likely duplicates.

Possible inputs:

- company
- normalized title
- normalized city
- event date
- venue

Example concept:

Company + Title + City + Event Date + Venue

This should not be treated as a perfect identity.

Two genuine opportunities can sometimes have similar values.

Therefore:

Fingerprint
→ candidate duplicate

Then:

Deduplication logic
→ final decision

---

## 24. URL Uniqueness

Normalized URLs should be indexed.

The same source page should not be repeatedly stored as a completely new source record simply because tracking parameters changed.

The system should keep:

Original URL

and

Normalized URL

separately when useful.

---

## 25. Indexing Strategy

Indexes should be based on actual query patterns.

Important initial indexes may include:

- job_opportunity.city
- job_opportunity.normalized_city
- job_opportunity.event_date
- job_opportunity.freshness_status
- job_opportunity.walk_in_status
- job_opportunity.company_id
- job_opportunity.created_at
- job_opportunity.last_seen_at
- job_opportunity_source.normalized_url
- job_source.source_type
- discovery_task.status
- discovery_task.created_at

Composite indexes can be introduced after observing real query patterns.

Do not create indexes for every column without evidence.

---

## 26. Search Query Pattern

A typical V1 query may be:

Active walk-in opportunities
+
matching city
+
matching experience
+
matching technology
+
upcoming event date

The database should be optimized for these filters.

For example:

freshness_status = ACTIVE

walk_in_status = WALK_IN

normalized_city = Bengaluru

event_date >= today

Skill matching can be handled through the relationship table initially.

---

## 27. Pagination

The opportunity API should support pagination.

Example:

GET /api/v1/opportunities

Possible parameters:

- page
- size
- city
- role
- skill
- experience
- date range
- freshness
- sort

The API should avoid returning an unlimited number of records.

Pagination becomes increasingly important as the database grows.

---

## 28. Sorting

The initial database query can support basic sorting such as:

- event date
- created date
- freshness

Final relevance ranking should remain part of the ranking/search layer.

The database should not contain complex ranking logic unnecessarily.

---

## 29. Timestamps

Important domain records should maintain timestamps.

Common fields:

- created_at
- updated_at

Discovery-related records may also require:

- discovered_at
- fetched_at
- first_seen_at
- last_seen_at
- expires_at

Timestamps should be generated consistently by the backend.

The system should use a consistent time standard internally.

---

## 30. Soft Deletion and Expiration

An expired job is not necessarily the same as a deleted job.

For example:

An opportunity can become:

EXPIRED

while still being retained for:

- history
- debugging
- source analysis
- duplicate detection
- quality metrics

Therefore, V1 should prefer status-based expiration over immediate physical deletion.

Retention policies can be introduced later.

---

## 31. Data Retention

Not all data needs to be stored forever.

Future retention policies may apply to:

- old raw documents
- failed fetch records
- expired opportunities
- discovery task logs

For V1, retention should remain simple.

Do not introduce a complicated archival system before actual data volume requires it.

---

## 32. Transaction Boundaries

Database transactions should be used around operations that require consistency.

Examples:

- creating or updating an opportunity
- linking an opportunity to a company
- linking skills
- updating source relationships

Long-running web fetching should not happen inside a database transaction.

Bad approach:

Start DB transaction
→ make HTTP request
→ wait
→ parse page
→ save result
→ commit

Better approach:

Fetch externally
→ process
→ validate
→ start short DB transaction
→ persist result
→ commit

This keeps database connections available for other operations.

---

## 33. Concurrent Processing

Multiple discovery workers may process related URLs at the same time.

The database design should therefore handle concurrent writes safely.

Useful mechanisms:

- unique constraints
- optimistic locking where required
- idempotent updates
- transactions
- careful upsert/update logic

The application should not assume that only one worker will ever process a source.

---

## 34. Migration Strategy

Database schema changes should be managed through versioned migrations.

For example:

V1:
Initial schema

V2:
Add source quality

V3:
Add verification fields

V4:
Add additional job type

The exact migration tool can be selected during backend implementation.

The important principle is:

**Never depend on manually modifying the production database schema.**

Schema changes should be reproducible.

---

## 35. Entity and DTO Separation

JPA entities should represent persistence concerns.

API DTOs should represent API contracts.

The system should avoid exposing JPA entities directly from controllers.

Example:

Database Entity
→ Service
→ Response DTO
→ API

This prevents database structure from becoming tightly coupled to the frontend API.

---

## 36. Initial Schema Complexity

The V1 database should remain intentionally simple.

Do not create tables for every possible future feature.

For example, V1 does not require:

- users
- subscriptions
- notifications
- resumes
- applications
- recruiter accounts
- payments
- recommendation profiles

These can be introduced when the corresponding product features are implemented.

---

## 37. Future User Model

When user accounts are introduced, the database can expand with tables such as:

- users
- saved_searches
- notifications
- user_preferences
- application_tracking

These should reference the existing opportunity model instead of replacing it.

---

## 38. Future Search Infrastructure

PostgreSQL is sufficient for initial V1 search.

If the opportunity dataset becomes large or search requirements become more advanced, a dedicated search engine can be introduced.

Possible future architecture:

PostgreSQL
→ source of truth

Search Index
→ optimized discovery/search experience

The search index should be treated as a derived representation, not the primary source of truth.

---

## 39. Future Caching

Redis may be introduced later for:

- frequently requested searches
- API caching
- rate limiting
- temporary discovery state
- distributed coordination

Redis is not required for the first version.

Caching should be introduced based on actual performance requirements.

---

## 40. Database Scaling Direction

Initial:

Spring Boot
→ PostgreSQL

Later:

Multiple API instances
→ PostgreSQL

Then, if required:

API instances
→ Connection Pool
→ PostgreSQL Primary
→ Read Replicas

Other supporting infrastructure can later include:

- Redis
- Search Index
- Object Storage

The database architecture should evolve based on real workload rather than anticipated scale alone.

---

## 41. Conceptual Data Model

The V1 conceptual model is:

Company
|
| 1
|
| N
v
JobOpportunity
|
| N
|
| N
v
JobOpportunitySource
|
| N
|
| 1
v
JobSource
|
| 1
|
| N
v
RawDocument

JobOpportunity
|
| N
|
| N
v
JobOpportunitySkill
|
| N
|
| 1
v
Skill

DiscoveryTask
|
| discovers/processes
v
Candidate URLs / Job Opportunities

This is a logical model, not a final physical schema.

---

## 42. Example Opportunity Lifecycle

A page is discovered:

Search Provider
→ URL

URL is fetched:

RawDocument

Walk-in information is detected:

WALK_IN

Job information is extracted:

Java Backend Developer
Bengaluru
3-5 years

Data is normalized:

Bangalore
→ Bengaluru

The opportunity is validated:

VALID

Duplicate check:

Existing opportunity found

The source is linked:

JobOpportunitySource

Freshness:

ACTIVE

Ranking:

High relevance

The opportunity is persisted:

PostgreSQL

The frontend retrieves it:

GET /api/v1/opportunities

---

## 43. Database Responsibility

The database is responsible for:

- durable storage
- relationships
- consistency
- constraints
- indexing
- querying
- persistence of normalized domain data
- tracking source relationships
- tracking freshness
- supporting idempotency

The database is not responsible for:

- web crawling
- HTML parsing
- walk-in classification
- query generation
- ranking algorithms
- HTTP fetching
- frontend logic

Those responsibilities belong to the backend application layers.

---

## 44. Final V1 Database Direction

The initial database architecture is:

PostgreSQL
|
+-- Companies
|
+-- Job Opportunities
|   |
|   +-- Skills
|   |
|   +-- Sources
|
+-- Job Sources
|
+-- Raw Documents
|
+-- Discovery Tasks

The database should remain the reliable source of truth while the backend discovery pipeline performs:

Discovery
→ Extraction
→ Normalization
→ Validation
→ Deduplication
→ Freshness
→ Ranking
→ Persistence

The main goal is to create a clean, reliable data foundation that can support V1 today and expand into a broader job discovery platform later.
