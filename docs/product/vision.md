# Product Vision

## Overview

Job Discovery Platform is a product designed to help people discover relevant and trustworthy hiring opportunities from across publicly accessible sources on the internet.

The platform will automatically discover, understand, organize, verify, and surface job-related opportunities based on what the user is looking for.

The initial focus is on solving one specific problem well:

> Discovering upcoming walk-in drives for a given city, role, technology, and experience range.

Walk-in discovery is the first use case, not the final scope of the platform.

---

## The Problem

Job opportunities are scattered across many different sources.

Candidates often need to:

- Search multiple websites and platforms
- Use different keywords for the same role
- Check whether a walk-in drive is still upcoming
- Identify the correct location and date
- Deal with duplicate postings
- Determine whether a posting is genuine or outdated
- Track opportunities from specific companies
- Repeatedly perform the same searches

This creates a noisy and time-consuming job discovery process.

The platform aims to reduce this effort by bringing relevant opportunities together in one simple and trustworthy experience.

---

## Vision

Build a simple and reliable job discovery platform that finds relevant hiring opportunities from across the web and presents them to users with useful context, freshness, and source transparency.

The long-term vision is not to become another large job listing website.

Instead, the platform aims to become a **discovery and intelligence layer for job opportunities**.

---

## Product Principles

### 1. Relevance Over Quantity

The goal is not to show the maximum number of job postings.

The goal is to show the opportunities that are actually relevant to the user's search.

---

### 2. Trust Over Noise

Every opportunity should have identifiable sources and, where possible, verification signals.

The platform should never present uncertain information as confirmed information.

---

### 3. Freshness Matters

Job opportunities change quickly.

The platform should detect expired opportunities, periodically re-check important information, and prioritize recently verified opportunities.

---

### 4. Simple User Experience

The user should not need to understand how the platform collects or processes information.

The experience should remain simple:

**Search → Relevant Opportunities → Details → Apply**

---

### 5. Source Transparency

The platform should make it clear where an opportunity was discovered.

Users should be able to access the original source whenever possible.

---

### 6. Source Independence

The platform should not depend on a single website or data provider.

The discovery architecture should support multiple sources and allow new sources to be added without redesigning the core system.

---

### 7. Scalable by Design

The initial implementation should remain simple enough to develop and operate efficiently.

At the same time, the architecture should allow the platform to scale as:

- Number of users increases
- Number of sources increases
- Number of cities increases
- Search volume increases
- Data volume increases
- More job types are introduced

---

## Initial Use Case

### V1 — Walk-in Discovery

A user can provide information such as:

- City
- Technology or role
- Experience range
- Date range

The platform discovers relevant publicly accessible pages and identifies upcoming walk-in opportunities.

The system then:

1. Discovers candidate sources
2. Fetches publicly accessible content
3. Identifies potential walk-in opportunities
4. Extracts structured information
5. Normalizes the information
6. Validates the extracted data
7. Detects duplicate opportunities
8. Tracks freshness
9. Stores the resulting opportunity
10. Makes it searchable through the backend API

---

## Long-Term Direction

Walk-in discovery is only the first problem the platform will solve.

The underlying discovery engine should eventually support other types of hiring information, including:

- Regular job postings
- Company-specific hiring
- Hiring drives
- Recruitment events
- Technology-specific opportunities
- Location-specific opportunities
- Personalized job discovery
- Company tracking
- Job alerts

These capabilities should be added without changing the fundamental discovery architecture.

---

## Long-Term Product Experience

A user should eventually be able to express an intent such as:

> "Show me Java backend opportunities in Bengaluru."

or:

> "Show me upcoming hiring opportunities at Amazon."

or:

> "Notify me when a new Spring Boot opportunity appears in Delhi NCR."

The platform should determine where relevant information can be found, collect it, evaluate it, and present the most useful results.

---

## What This Product Is Not

The platform is not intended to:

- Become a simple job-posting directory
- Compete only on the number of listings
- Depend entirely on one job portal
- Show large amounts of unverified or duplicated data
- Hide the original source of information
- Force users through a complicated search experience

---

## Success Criteria

The product should ultimately be judged by:

- Relevance of discovered opportunities
- Accuracy of extracted information
- Freshness of listings
- Duplicate reduction
- Source transparency
- Search quality
- User usefulness
- Reliability of the discovery pipeline

The number of listings alone is not a primary measure of success.

---

## Product Direction

Start narrow.

Solve walk-in discovery properly.

Build the discovery engine as a reusable foundation.

Then expand into broader job discovery capabilities without compromising simplicity, trust, and relevance.
