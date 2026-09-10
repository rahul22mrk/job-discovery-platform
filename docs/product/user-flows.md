# User Flows

## V1 - Walk-in Search

The main V1 user flow:

User
↓
Select City
↓
Select Technology / Role
↓
Select Experience
↓
Select Date Range
↓
Search
↓
System discovers opportunities
↓
Relevant walk-in results
↓
User checks details
↓
User opens original source
↓
Apply / Attend

## Search Inputs

V1 will support the following inputs:

- City
- Technology / Role
- Experience
- Date Range

Technology and role can be selected from predefined options to improve search quality.

Users should also be able to enter a custom keyword if the required option is not available.

Example:

City: Bengaluru
Technology: Java
Experience: 3-5 years
Date: Next 30 days

## Backend Discovery Flow

Search Request
↓
Create Discovery Task
↓
Generate Search Queries
↓
Search Multiple Sources
↓
Collect URLs
↓
Normalize URLs
↓
Remove Duplicate URLs
↓
Fetch Public Pages
↓
Extract Page Content
↓
Identify Walk-in Opportunities
↓
Extract Job Information
↓
Normalize Information
↓
Validate Information
↓
Identify Duplicate Opportunities
↓
Check Freshness
↓
Store / Update Opportunity
↓
Make Opportunity Available for Search

## Search Query Generation

The system should generate multiple search queries from a single user search.

Example:

City: Bengaluru
Technology: Java

Possible queries:

- "walk-in interview" "Bengaluru" "Java"
- "walk-in drive" "Bengaluru" "Java"
- "walkin interview" "Bengaluru" "Java"
- "walk-in recruitment" "Bengaluru" "Java"
- "Java developer" "walk-in" "Bengaluru"
- "Java backend" "walk-in" "Bengaluru"

The query generation strategy should be configurable so that it can be improved later.

## URL Discovery

Search providers will return candidate pages.

The system should collect:

- URL
- Title
- Description or snippet
- Source
- Discovery time

The same URL may appear in multiple search results.

Duplicate URLs should be removed before page processing.

## Page Fetching

For each unique URL:

URL
↓
HTTP Request
↓
HTML Response
↓
Content Extraction
↓
Raw Document

The system should handle common failures such as:

- Timeout
- Connection failure
- HTTP errors
- Invalid pages
- Empty content
- Unsupported content

A failure for one page should not stop the complete discovery process.

## Walk-in Detection

Not every discovered page will be a walk-in opportunity.

The system should first determine whether the page is relevant.

Possible signals include:

- walk-in
- walk in
- walkin
- walk-in interview
- walk-in drive
- recruitment drive
- direct interview
- open interview

The first version can use rule-based detection.

A more advanced classification system can be added later.

## Information Extraction

After a page is identified as a potential walk-in, the system should extract useful information.

Possible fields:

- Company
- Job Title
- Technology / Skills
- City
- Venue
- Event Date
- Start Time
- End Time
- Experience
- Description
- Application Method
- Source URL
- Source Name

Not every source will contain every field.

Missing information should remain missing rather than being guessed.

## Data Normalization

Different sources may use different formats.

Examples:

Bangalore
Bengaluru

These may refer to the same city.

Similarly:

Java Developer
Java Backend Developer
Java Engineer

may have related meanings.

The backend should normalize common values where appropriate.

Normalization should happen before validation and deduplication.

## Validation

Extracted information should be validated before becoming an active opportunity.

### Date

If the event date has already passed, the opportunity should not be treated as upcoming.

### City

The opportunity should match the requested location.

City variations should be handled where appropriate.

### Walk-in

There should be enough evidence that the opportunity is actually a walk-in or hiring drive.

### Data

Important fields should be checked for invalid or inconsistent values.

The system should prefer missing information over guessed information.

## Duplicate Opportunity Detection

Different URLs can represent the same hiring opportunity.

For example:

- Company Website
- LinkedIn
- Job Portal
- Recruitment Website

may all refer to the same opportunity.

The system should try to identify these as one opportunity.

Possible matching fields:

- Company
- Role
- City
- Event Date
- Venue

The exact matching strategy can become more advanced later.

Users should normally see one opportunity instead of multiple duplicate results.

Source information can still be stored internally.

## Freshness

Every opportunity should have freshness information.

Important fields may include:

- discoveredAt
- lastCheckedAt
- eventDate
- status

Example:

Discovered: 10 Sep 2026
Last Checked: 10 Sep 2026
Event Date: 15 Sep 2026
Status: UPCOMING

After the event date:

Status: EXPIRED

Expired opportunities should not appear in active upcoming results.

## Source Transparency

Each opportunity should keep information about where it was discovered.

Example:

Company: ABC Technologies
Role: Java Developer
Source: Company Website
Original URL: Source URL

If the same opportunity is found on multiple sources, the system can maintain all known sources.

Users should be able to open the original source.

## Trust Flow

The system should collect signals that can help determine how reliable an opportunity is.

Possible signals:

- Company source available
- Multiple sources found
- Event date available
- Venue available
- Application link available
- Recently checked
- Source still accessible

The product should not label an opportunity as verified unless the available evidence supports that status.

If information is uncertain, the user should be informed.

## Result Flow

After processing, the user should receive relevant opportunities.

Example:

Java Developer
ABC Technologies

Bengaluru
15 Sep 2026
10 AM - 2 PM

3-5 years

Source: Company Website
Last checked: 2 hours ago

View Details

## Search Result Ranking

Results should eventually be ranked based on relevance.

Possible ranking signals:

- Technology match
- Role match
- City match
- Experience match
- Date relevance
- Source quality
- Freshness
- Availability of important information

The ranking system should prioritize useful opportunities rather than simply returning results in discovery order.

## User Opens Opportunity

When the user opens an opportunity:

Opportunity
↓
Details
↓
Source Information
↓
Original Source
↓
Apply / Attend

The platform should not replace the original source when the original source is required for application.

## Future Company Search

The same discovery system should later support:

- Company + Role
- Company + Technology
- Company + City

Example:

Company: Amazon
Technology: Java
City: Bengaluru

This should use the same discovery and processing pipeline.

## Future Saved Search

Users may eventually save searches.

Example:

Java
Bengaluru
Walk-in

The system can periodically check for new matching opportunities.

## Future Notifications

Saved searches can later be connected to notifications.

Flow:

New Opportunity
↓
Matches Saved Search
↓
Check Opportunity Status
↓
Notify User

Notifications are outside V1.

## Error Handling

A failure in one part of the discovery process should not stop the complete system.

Example:

Source A → Failed
Source B → Success
Source C → Success

The system should continue processing successful sources.

Failures should be logged so they can be investigated later.

## Future Expansion

The user flow should eventually support other opportunity types:

- WALK_IN
- JOB_POSTING
- HIRING_DRIVE
- COMPANY_JOB

The underlying discovery process should remain reusable.

Only the classification and extraction requirements may change.

## Main Principle

The user experience should remain simple:

Search
↓
Relevant Opportunities
↓
Check Details
↓
Check Source
↓
Apply / Attend

The discovery, extraction, validation, deduplication, and freshness checks should happen in the background.
