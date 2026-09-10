# Frontend Project Guidelines

## 1. Purpose

This document defines the rules for developing the frontend of the Job Discovery Platform.

These rules apply to human developers as well as AI coding assistants such as ChatGPT, Claude, Cursor, Copilot, and similar tools.

The purpose is to keep the frontend:

- Simple
- Maintainable
- Consistent
- Accessible
- Performant
- Secure
- Easy to extend
- Properly integrated with the backend

The frontend must follow the product and backend architecture already defined in this repository.

AI tools must read and follow these guidelines before making frontend changes.

---

# 2. Frontend Technology Baseline

The frontend implementation uses:

- React
- TypeScript
- Modern JavaScript
- HTML5
- CSS / project-selected styling solution
- REST APIs exposed by the backend

The exact libraries used for routing, styling, API requests, state management, and testing must be selected based on actual project requirements.

Do not add a library simply because it is popular.

Prefer existing project dependencies over introducing new ones.

---

# 3. Core Principles

The frontend should follow these principles:

1. Keep components small and focused.
2. Keep business logic outside UI components where practical.
3. Keep API communication outside presentation components.
4. Prefer simple solutions over complex abstractions.
5. Keep state minimal.
6. Avoid duplicated or derived state.
7. Follow one-way data flow.
8. Make loading, error, empty, and success states explicit.
9. Treat API data as untrusted input.
10. Build accessible interfaces by default.
11. Reuse components when there is real reuse.
12. Do not optimize prematurely.
13. Do not put backend/discovery logic in the frontend.
14. Do not introduce architecture without a real requirement.
15. Preserve existing API contracts.

---

# 4. Product Boundary

The frontend is responsible for:

- Collecting user search input
- Calling backend APIs
- Showing search progress
- Showing job opportunities
- Showing filters and sorting
- Showing job details
- Showing source information
- Showing freshness and trust information
- Handling loading, error, and empty states
- Providing a simple and usable experience

The frontend is NOT responsible for:

- Discovering jobs from the internet
- Generating search-engine queries
- Crawling websites
- Fetching external job websites directly
- Extracting job information from HTML
- Detecting walk-in opportunities
- Deduplicating jobs
- Determining job freshness
- Ranking opportunities using backend ranking logic
- Storing backend business data locally as the source of truth

The backend owns discovery and job intelligence.

The frontend consumes backend APIs.

---

# 5. Architecture

Use a clear separation between:

```text
UI
 ↓
Feature / Page Logic
 ↓
API / Service Layer
 ↓
Backend REST API
