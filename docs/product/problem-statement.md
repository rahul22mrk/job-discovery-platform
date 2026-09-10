# Problem Statement

## Problem

Job seekers often have to search across multiple websites, job portals, company career pages, social platforms, and other public sources to find relevant hiring opportunities.

This becomes particularly difficult for walk-in drives because the information is often:

- Distributed across multiple sources
- Published using inconsistent terminology
- Difficult to search systematically
- Reposted across multiple platforms
- Missing important details
- Updated without clear visibility
- Left online even after the opportunity has expired

As a result, candidates spend significant time searching, verifying, filtering, and comparing opportunities instead of focusing on applying to the right ones.

---

## The Specific V1 Problem

The first problem the product will solve is:

> **Given a city and a job-related search intent, automatically discover upcoming walk-in opportunities from publicly accessible sources across the internet and present the relevant opportunities in a clean, trustworthy, and searchable format.**

For example:

A user wants:

- City: Bengaluru
- Technology: Java
- Experience: 3–5 years
- Time range: Next 30 days

Today, the user may need to perform several searches across different sources and manually determine which results are:

- Actually walk-in opportunities
- Relevant to Java
- Located in Bengaluru
- Suitable for their experience
- Still upcoming
- Genuine or sufficiently trustworthy
- Duplicates of another posting

The platform should automate as much of this process as reasonably possible.

---

## Why Existing Search Is Not Enough

A conventional search engine can find pages containing keywords, but it does not necessarily understand that multiple pages may represent the same hiring event.

For example, the following may all describe the same opportunity:

- A company website
- A job portal
- A social media post
- A recruitment page
- A reposted article

A candidate should ideally see one useful opportunity rather than several duplicated results.

Similarly, a page containing the words "walk-in interview" does not necessarily mean that an upcoming walk-in is available.

The platform therefore needs to go beyond keyword search.

---

## Core Problems to Solve

### 1. Discovery

Find relevant publicly accessible pages across multiple sources.

---

### 2. Classification

Determine whether a discovered page actually represents a relevant hiring opportunity.

For V1, the primary classification is:

**Is this an upcoming walk-in opportunity?**

---

### 3. Information Extraction

Extract useful structured information from unstructured web content.

Examples:

- Company
- Job title
- Technology/skills
- City
- Venue
- Event date
- Start time
- End time
- Experience requirements
- Application method
- Original source

---

### 4. Validation

Determine whether the extracted information is sufficiently reliable.

Examples:

- Is the event date in the future?
- Does the location match the requested city?
- Is there evidence that this is a walk-in?
- Is the source still accessible?
- Are important fields missing or contradictory?

---

### 5. Deduplication

Identify when multiple sources refer to the same opportunity.

The system should consolidate duplicate information instead of presenting it as separate opportunities.

---

### 6. Freshness

Hiring information can become outdated quickly.

The platform should track when information was discovered and when it was last verified.

Expired opportunities should not continue to appear as active opportunities.

---

### 7. Relevance

The system should prioritize opportunities that closely match the user's search intent.

For example:

**Search:**

Java + Bengaluru + 3–5 years

should prioritize:

Java Backend Developer + Bengaluru + 3–5 years

over a generic software job with no Java requirement.

---

### 8. Source Transparency

The user should be able to understand where an opportunity came from and access the original source.

The platform should not hide the origin of discovered information.

---

## Desired Outcome

The desired outcome is a workflow that looks like:

User Search

↓

Internet Discovery

↓

Candidate Pages

↓

Classification

↓

Information Extraction

↓

Validation

↓

Deduplication

↓

Freshness Evaluation

↓

Relevant Opportunities

↓

Original Source / Apply

The user should not need to manually perform these intermediate steps.

---

## Product Quality Requirements

The platform should prioritize:

### Accuracy

Extracted information should be as accurate as possible.

### Freshness

Old or expired opportunities should be identified and removed from active results.

### Relevance

Search results should match the user's intent rather than simply matching keywords.

### Transparency

The source of information should remain visible.

### Reliability

Failure of one source should not prevent the overall discovery system from functioning.

### Scalability

The architecture should support additional sources, cities, job types, and users in the future.

---

## V1 Boundary

V1 will focus specifically on:

**Upcoming Walk-in Opportunities**

The following are intentionally outside the initial scope:

- General job aggregation
- Full company tracking
- Personalized recommendations
- Notifications
- Mobile applications
- Advanced AI recommendations
- Paid features
- Large-scale microservice infrastructure

These may be considered after the core discovery problem is solved reliably.

---

## Future Expansion

The same underlying discovery and processing system should eventually support:

- Regular job postings
- Company-specific hiring
- Recruitment drives
- Hiring events
- Technology-specific job discovery
- Location-specific job discovery
- Saved searches
- Company tracking
- Job alerts
- Personalized opportunity discovery

The V1 implementation should therefore avoid architectural decisions that make these future capabilities unnecessarily difficult to introduce.

---

## Problem Success Definition

V1 will be considered successful when the platform can reliably perform the following flow:

**City + Job Intent**

↓

**Discover relevant public web sources**

↓

**Identify genuine upcoming walk-in opportunities**

↓

**Extract useful structured information**

↓

**Remove duplicates**

↓

**Filter expired opportunities**

↓

**Return clean and relevant results**

The objective is not to maximize the number of discovered pages.

The objective is to maximize the usefulness of the final opportunities shown to the user.
