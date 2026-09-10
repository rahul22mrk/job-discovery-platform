# Extraction Architecture

## Purpose

The extraction layer converts publicly available webpage content into structured job information.

Its responsibility is to answer:

- Is this page related to a job?
- Is it a walk-in opportunity?
- What company is hiring?
- What role is available?
- Where is the opportunity?
- When is the walk-in?
- What experience is required?
- What skills are required?
- How can the candidate apply?

The extraction layer must preserve source information and must never invent missing data.

---

## 1. Extraction Position in Pipeline

Extraction happens after fetching and before normalization.

The flow is:

Page Fetching
→ Raw Document
→ Content Processing
→ Structured Data Extraction
→ Job Extraction
→ Walk-in Detection
→ Normalization
→ Validation

Extraction produces candidate data.

Normalization makes that data consistent.

Validation decides whether the candidate data is good enough to become a real opportunity.

---

## 2. Extraction Responsibilities

The extraction module is responsible for:

- Reading raw webpage content
- Cleaning HTML
- Reading structured data
- Extracting job fields
- Detecting walk-in signals
- Identifying event information
- Extracting skills
- Extracting experience
- Producing extraction confidence
- Reporting extraction failures

It is not responsible for:

- URL discovery
- HTTP fetching
- Persistence
- Global deduplication
- Final ranking
- Frontend formatting

---

## 3. Raw Input

The extraction layer receives a `RawDocument`.

Possible information:

- source URL
- normalized URL
- source name
- content type
- HTTP status
- raw content
- content hash
- fetched timestamp

Example:

sourceUrl:
https://example.com/jobs/java-walkin

contentType:
text/html

rawContent:
HTML page content

The extraction layer should not need to know how the page was discovered.

---

## 4. Content Processing

Raw HTML normally contains a lot of irrelevant content.

The content-processing stage should reduce noise.

Potential elements to remove:

- script
- style
- navigation
- advertisements
- tracking elements
- unrelated widgets
- repeated footer content

The goal is to produce clean text while preserving useful job information.

---

## 5. Structured Data First

The first extraction strategy should inspect structured information.

Possible formats:

- JSON-LD
- Schema.org JobPosting
- OpenGraph
- HTML metadata

For job pages, structured data may contain:

- title
- hiring organization
- description
- employment type
- job location
- date posted

Structured data should be preferred when available.

However, it must still be validated.

---

## 6. JSON-LD Extraction

Many webpages embed JSON-LD inside:

script elements

with:

application/ld+json

The extractor should:

1. Find JSON-LD blocks
2. Parse valid JSON
3. Identify JobPosting objects
4. Extract supported fields
5. Ignore unrelated schemas
6. Continue with other extraction methods if data is incomplete

A page can contain multiple JSON-LD objects.

The extractor should not assume the first object is always the job.

---

## 7. JobPosting Detection

The extractor should recognize JobPosting structured data when available.

Useful fields include:

- title
- description
- hiringOrganization
- jobLocation
- employmentType
- datePosted
- validThrough
- baseSalary where available

The system should preserve only fields that are actually present.

---

## 8. Generic DOM Extraction

If structured data is missing or incomplete, the extractor can inspect the DOM.

Potential signals:

- page title
- heading elements
- labels
- tables
- lists
- paragraphs
- metadata
- buttons
- links

Common job-page labels include:

- Job Title
- Role
- Experience
- Location
- Walk-in Date
- Interview Date
- Venue
- Contact
- Apply

The extractor should use multiple signals.

---

## 9. Text Pattern Extraction

Some websites contain mostly unstructured text.

Text-pattern extraction can identify useful information.

Examples:

"2-5 years"

"Minimum 3 years"

"Walk-in interview on 15 September"

"Reporting time: 10 AM"

"Venue: Bengaluru"

Patterns should be carefully designed and tested.

A keyword appearing in unrelated content should not automatically create a field.

---

# Walk-in Detection

## 10. Walk-in Detection

Walk-in detection is a core V1 capability.

The extractor should classify pages as:

- WALK_IN
- NOT_WALK_IN
- UNCERTAIN

The classification should be based on evidence.

---

## 11. Walk-in Signals

Possible positive signals:

- walk-in interview
- walk in interview
- walk-in drive
- walk in drive
- walk-in recruitment
- walkin interview
- walkin drive
- walk-in hiring
- attend interview
- reporting time
- interview venue
- interview date
- walk-in venue

Multiple signals should increase confidence.

---

## 12. Negative Signals

Some pages may contain the word "walk-in" without representing an actual walk-in opportunity.

Examples:

- old news article
- generic recruitment advice
- unrelated blog content
- discussion mentioning walk-in interviews
- expired content

Negative signals and context should therefore be considered.

The system should not classify a page as a walk-in solely because one keyword appears.

---

## 13. Walk-in Evidence

The extraction result should retain evidence.

Conceptually:

walkInStatus:
WALK_IN

walkInEvidence:
"Walk-in interview"

walkInConfidence:
0.94

The exact representation can change during implementation.

The important point is that the system should be able to explain why a page was classified as a walk-in.

---

## 14. Walk-in Event Extraction

A walk-in opportunity should attempt to extract:

- event date
- start time
- end time
- reporting time
- venue
- address
- contact information

Example:

Interview Date:
15 September 2026

Reporting Time:
10:00 AM

Venue:
Bengaluru Office

These fields are especially important because the usefulness of a walk-in opportunity depends heavily on event information.

---

## 15. Date Extraction

Dates can appear in many formats.

Examples:

15 September 2026

15/09/2026

15-09-2026

September 15, 2026

The extractor should convert recognized dates into a consistent internal representation.

Ambiguous dates should not be guessed.

---

## 16. Relative Dates

Some pages may contain:

Tomorrow

This Saturday

Next Monday

Such dates require the page's publication/fetch context and timezone to resolve correctly.

If the date cannot be safely resolved, it should remain uncertain.

The system should not invent a date.

---

## 17. Time Extraction

Possible formats:

10 AM

10:00 AM

10 AM - 4 PM

10:00-16:00

The extractor should normalize time into the backend's standard representation.

Reporting time and interview start time should remain distinguishable when both exist.

---

## 18. Venue Extraction

Potential venue information may include:

- office name
- building
- street
- locality
- city
- address
- landmark

Example:

Venue:
ABC Technologies
Electronic City
Bengaluru

The original text should be retained when useful.

The extractor should not construct an address from assumptions.

---

# Job Extraction

## 19. Company Extraction

Possible company signals:

- hiring organization in structured data
- company heading
- page title
- job description
- company logo metadata
- source domain

The domain name may be used as a supporting signal but should not automatically become the company name.

---

## 20. Job Title Extraction

Possible signals:

- JobPosting.title
- H1
- page title
- job heading
- structured metadata

The extractor should avoid including irrelevant page text such as:

"Apply Now"

or:

"Latest Jobs"

as the job title.

---

## 21. Description Extraction

The description should contain the actual job information where possible.

It may include:

- responsibilities
- requirements
- eligibility
- hiring information
- interview details

Unrelated navigation and advertisements should be excluded.

---

## 22. Experience Extraction

The extractor should identify experience requirements.

Examples:

Fresher

0-2 years

2-5 years

3+ years

Minimum 3 years

Experience not mentioned

The extracted values are passed to normalization.

---

## 23. Skill Extraction

Potential skills include:

- Java
- Spring Boot
- Microservices
- Kafka
- SQL
- AWS
- Docker

Skill extraction can use:

- structured fields
- job description
- known technology dictionary
- source-specific information
- semantic extraction in future

The initial implementation should prefer deterministic matching.

---

## 24. Location Extraction

Potential sources:

- JobPosting.jobLocation
- page metadata
- job heading
- address
- event venue
- text patterns

The extractor should distinguish between:

Job Location

and:

Walk-in Venue

They can be the same but are not always identical.

---

## 25. Job Type Extraction

Potential job types:

- FULL_TIME
- PART_TIME
- CONTRACT
- INTERNSHIP
- TEMPORARY
- UNKNOWN

V1 primarily focuses on walk-in opportunities.

The job type should remain optional when not provided.

---

# Extraction Result

## 26. Extraction Result Model

The extraction layer should return an internal result.

Conceptually:

ExtractionResult

Possible fields:

- company
- title
- description
- location
- country
- jobType
- minimumExperience
- maximumExperience
- skills
- eventDate
- eventStartTime
- eventEndTime
- reportingTime
- venue
- applicationUrl
- applicationMethod
- walkInStatus
- walkInEvidence
- extractionConfidence
- extractionWarnings

The exact Java model can be finalized during implementation.

---

## 27. Extraction Confidence

Each extraction result can have a confidence level.

Possible values:

- HIGH
- MEDIUM
- LOW

Or a numeric score can be used internally.

Confidence should represent how reliably the information was extracted.

It should not mean that the job itself is trustworthy.

---

## 28. Field-Level Confidence

In future, confidence can be tracked per field.

Example:

company:
HIGH

title:
HIGH

eventDate:
MEDIUM

venue:
LOW

This can help validation and review.

V1 can use a single overall extraction confidence if that keeps implementation simple.

---

## 29. Missing Information

Missing information should remain missing.

Example:

If the source does not provide:

event time

the system should store:

event time = unknown

It should not infer:

10:00 AM

from a generic assumption.

---

## 30. Conflicting Information

Different parts of a page may contain conflicting information.

Example:

Structured data:

Event date:
15 September

Page text:

Event date:
16 September

The extraction layer should record the conflict.

Final resolution can be handled by validation or source-specific rules.

The system should not silently hide contradictory data.

---

# Extraction Strategy

## 31. Extraction Order

Recommended order:

1. Structured data
2. Source-specific parser
3. DOM extraction
4. Text patterns
5. Optional AI extraction

The system should combine results where necessary.

---

## 32. Source-Specific Parser

Important sources may eventually receive dedicated parsers.

Example:

CompanyCareerPageParser

GreenhouseParser

LeverParser

The parser should only handle source-specific structure.

The final result should still pass through the common extraction and validation flow.

---

## 33. Generic Extraction

Unknown websites should use generic extraction.

Generic extraction should attempt to identify:

- title
- company
- location
- experience
- skills
- event information
- walk-in signals

It should be conservative.

A partially extracted result is better than confidently incorrect data.

---

# AI-Assisted Extraction

## 34. AI as Fallback

AI should not be required for every page.

Preferred flow:

Structured Data
→ Source Parser
→ DOM/Text Rules
→ AI Fallback if needed

This reduces:

- cost
- latency
- unpredictability

---

## 35. AI Use Cases

AI can be useful for:

- ambiguous walk-in detection
- complex page extraction
- semantic job title extraction
- skill extraction
- conflicting text interpretation
- duplicate similarity
- difficult eligibility interpretation

AI should produce structured output that still passes validation.

---

## 36. AI Output Validation

AI output must not be treated as trusted data.

Example:

AI says:

eventDate = 2026-09-15

The backend should still verify that:

- the date exists in source content
- the date is plausible
- the page is relevant
- the event is not expired

AI can assist extraction.

The source remains the authority.

---

# Extraction Failure Handling

## 37. Invalid HTML

If HTML cannot be parsed:

- record extraction failure
- retain raw document metadata
- continue processing other URLs

One broken page must not stop discovery.

---

## 38. Unsupported Content

If the fetched content is unsupported:

- mark extraction as unsupported
- record the reason
- skip the page
- continue the discovery task

Examples:

- unsupported binary document
- unexpected content type
- oversized content

Support for additional formats can be added later.

---

## 39. Partial Extraction

A page may contain enough information to be useful even if some fields are missing.

Example:

Company:
Available

Title:
Available

City:
Available

Walk-in date:
Available

Venue:
Missing

This can still become a valid opportunity if the required validation rules are satisfied.

The system should not reject every result simply because optional fields are missing.

---

# Extraction Quality

## 40. Required vs Optional Fields

Required fields for a useful V1 opportunity may include:

- company
- title
- source URL
- walk-in evidence
- location
- event date or sufficient event information

Optional fields may include:

- venue
- event end time
- description
- salary
- application method
- contact person

The exact validation requirements can evolve based on real data.

---

## 41. Extraction Warnings

The extractor should be able to report warnings.

Examples:

- company not found
- event date uncertain
- venue missing
- multiple possible titles
- conflicting dates
- low walk-in confidence

Warnings are useful for validation and debugging.

---

## 42. Extraction Metrics

The backend should measure:

- pages processed
- pages successfully extracted
- pages partially extracted
- extraction failures
- structured data availability
- walk-in positive rate
- walk-in uncertain rate
- average extraction confidence
- fields frequently missing

These metrics help improve the extraction engine.

---

# Testing

## 43. Unit Tests

Test individual extraction rules.

Examples:

- Java experience extraction
- date extraction
- time extraction
- walk-in keyword detection
- company extraction
- title extraction

Each rule should have positive and negative cases.

---

## 44. Fixture-Based Tests

Real webpage samples can be stored as test fixtures where legally and operationally appropriate.

For each fixture:

Input:
HTML

Expected:
ExtractionResult

This makes parser changes safer.

---

## 45. Regression Tests

When an extraction bug is fixed, add a regression test.

Example:

Problem:

Walk-in date incorrectly extracted.

Fix:

Improve date parser.

Regression test:

Ensure the original problematic page is handled correctly.

This prevents future changes from reintroducing the bug.

---

# Extraction and Normalization Boundary

## 46. Extraction

Extraction answers:

"What does the source page say?"

Example:

Bangalore

3-5 years

Walk-in interview on 15 September

---

## 47. Normalization

Normalization answers:

"How should the platform represent it consistently?"

Example:

Bangalore
→ Bengaluru

3-5 years
→ min = 3
→ max = 5

The two responsibilities should remain separate.

---

# Extraction and Validation Boundary

## 48. Extraction

Extraction says:

"The page appears to contain a walk-in interview."

---

## 49. Validation

Validation says:

"This information is sufficient and consistent enough to become a user-facing opportunity."

This separation is important.

Extraction should not decide final product validity.

---

# Extraction and Deduplication Boundary

## 50. Extraction

Extraction produces:

Company
Title
City
Date
Venue

---

## 51. Deduplication

Deduplication compares this opportunity against other opportunities.

It determines whether:

"This is the same opportunity already discovered elsewhere."

The extraction module should not perform global deduplication.

---

# Performance

## 52. Extraction Performance

Extraction should be efficient because a discovery task may process many pages.

Prefer:

- structured parsing
- lightweight DOM processing
- compiled/reusable patterns
- limited text scanning
- bounded AI usage

Avoid expensive processing when a page is obviously irrelevant.

---

## 53. Early Filtering

Where practical, obvious non-job pages can be filtered early.

For example:

Page has no job-related signals
→ low priority or skip expensive extraction

Page contains strong job signals
→ continue processing

This helps reduce processing cost.

---

# Security

## 54. Untrusted Web Content

Webpage content is untrusted input.

The extractor must not execute arbitrary webpage scripts.

HTML should be treated as data.

Do not allow external webpage content to directly execute backend code.

---

## 55. Data Size Limits

The extraction system should have limits for:

- response size
- HTML size
- text size
- JSON-LD size
- number of extracted elements

This protects the backend from unexpectedly large pages.

---

# Scalability

## 56. Extraction Workers

V1 extraction can run inside the backend's controlled asynchronous worker pool.

Later, extraction can be separated into dedicated workers if volume increases.

Possible future flow:

Discovery Queue
→ Fetch Workers
→ Extraction Workers
→ Validation Workers

This is not required for V1.

---

## 57. Selective AI Scaling

If AI-assisted extraction is introduced, it should be used selectively.

Example:

1000 pages fetched

700:
deterministically extracted

200:
partially extracted

100:
ambiguous

AI can be used mainly for the ambiguous subset.

This reduces cost and latency.

---

# Final V1 Extraction Flow

The V1 extraction architecture is:

Raw Document
|
v
HTML / Content Cleaning
|
v
Structured Data
|
+----------------------+
| Data Available       |
|        |             |
|       Yes            No
|        |             |
v        |             v
Structured       Source / Generic
Extraction          Extraction
|                    |
+---------+----------+
          |
          v
      DOM / Text
      Extraction
          |
          v
    Walk-in Detection
          |
          v
     Job Extraction
          |
          v
   Extraction Result
          |
          v
      Confidence
          |
          v
     Normalization
          |
          v
      Validation

AI-assisted extraction can be introduced as a fallback before the final ExtractionResult when deterministic methods are insufficient.

---

# Core Design Principles

## 1. Never invent data

If the source does not say it, do not create it.

## 2. Source is the authority

Extract from the source and preserve evidence.

## 3. Deterministic first

Use structured data, parsers, DOM, and rules before AI.

## 4. AI is a fallback

Use AI where it provides meaningful value.

## 5. Conservative classification

Uncertain should remain uncertain.

## 6. Extraction is not validation

Extract first, validate later.

## 7. Extraction is not normalization

Preserve source values before normalizing them.

## 8. One page failure must not stop discovery

Every source and page should fail independently.

## 9. Keep the extractor modular

New source parsers should not require changes throughout the backend.

## 10. Measure extraction quality

The platform should continuously measure what it extracts correctly and what it misses.

---

# Final Responsibility

The extraction layer has one central responsibility:

**Convert messy public-web content into structured, traceable job information without inventing facts.**

The final pipeline remains:

Fetch
→ Extract
→ Detect Walk-in
→ Extract Job Data
→ Normalize
→ Validate
→ Deduplicate
→ Freshness
→ Rank
→ Store
