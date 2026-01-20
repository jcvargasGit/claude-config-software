---
name: spec-prd
description: PRD (Product Requirements Document) expert. Covers problem statement, scope, success metrics, requirements, and technical specifications. Use for writing PRDs or product specs.
model: opus
---

# PRD (Product Requirements Document) Skill

Apply these patterns when writing PRDs.

## Structure

```markdown
1. Overview
2. Problem Statement
3. Goals & Success Metrics
4. Scope
5. User Stories
6. Requirements
7. Design & UX
8. Technical Considerations
9. Test Plan
10. Timeline & Milestones
11. Open Questions
```

## Sections

### 1. Overview

Brief summary for stakeholders who won't read the full document.

```markdown
## Overview

**Feature:** Password Reset Flow
**Owner:** @username
**Status:** Draft | In Review | Approved
**Last Updated:** 2025-01-15

One paragraph summary of what we're building and why.
```

### 2. Problem Statement

```markdown
## Problem Statement

### Current State
How things work today and why it's insufficient.

### User Pain Points
- Specific problems users face
- Quantify with data if available

### Business Impact
- Cost of not solving this
- Opportunity cost
```

### 3. Goals & Success Metrics

```markdown
## Goals

### Primary Goal
Single, measurable objective.

### Success Metrics
| Metric | Current | Target | Timeframe |
|--------|---------|--------|-----------|
| Password reset completion rate | 45% | 80% | 30 days post-launch |
| Support tickets for login issues | 200/week | 50/week | 60 days post-launch |

### Non-Goals
Explicitly what we are NOT trying to achieve.
```

### 4. Scope

```markdown
## Scope

### In Scope
- Feature A
- Feature B

### Out of Scope
- Feature C (deferred to v2)
- Feature D (separate initiative)

### Dependencies
- External system X
- Team Y's API
```

### 5. User Stories

Reference the user-stories skill for format.

```markdown
## User Stories

### Epic: Password Reset

#### Story 1: Request Reset
As a user who forgot their password,
I want to request a reset link via email,
So that I can regain access to my account.

[Acceptance criteria...]
```

### 6. Requirements

```markdown
## Requirements

### Functional Requirements

| ID | Requirement | Priority | Notes |
|----|-------------|----------|-------|
| FR-1 | User can request reset via email | Must | |
| FR-2 | Reset link expires after 1 hour | Must | Security |
| FR-3 | User can request SMS reset | Should | Phase 2 |

### Non-Functional Requirements

| ID | Requirement | Target |
|----|-------------|--------|
| NFR-1 | Reset email delivered within 30 seconds | 95th percentile |
| NFR-2 | Reset page loads in under 2 seconds | 90th percentile |
| NFR-3 | System handles 100 concurrent reset requests | Load test |
```

### 7. Design & UX

```markdown
## Design

### User Flow
[Diagram or description of user journey]

### Wireframes
[Links to Figma/design files]

### Copy
| State | Message |
|-------|---------|
| Success | "Check your email for reset instructions" |
| Error: Invalid email | "We couldn't find an account with that email" |
| Error: Rate limited | "Too many attempts. Please try again in 15 minutes" |
```

### 8. Technical Considerations

```markdown
## Technical Considerations

### Architecture
High-level approach and key decisions.

### API Changes
New or modified endpoints.

### Data Model
Schema changes or new entities.

### Security
Authentication, authorization, data protection.

### Performance
Expected load, caching strategy, rate limiting.

### Monitoring
Key metrics to track, alerts to configure.
```

### 9. Test Plan

Define what should be tested and at which level using **black-box approach** - test through the API with HTTP inputs/outputs.

```markdown
## Test Plan

### Unit Tests (Mocked - Local)
Simulate failure scenarios through the handler:

| Scenario | Input | Expected Response |
|----------|-------|-------------------|
| Successful operation | POST /endpoint with valid data | 201 + success message |
| Invalid email format | POST /endpoint with email: "invalid" | 400 + "Invalid email" error |
| Service unavailable | POST /endpoint (mocked service down) | 503 + "Service unavailable" |
| Rate limited | POST /endpoint (rate limit exceeded) | 429 + "Too many requests" |

### Integration Tests (Real AWS - Ephemeral)
Test actual cloud service interactions (happy paths):

| Scenario | What to Verify |
|----------|----------------|
| Create resource | Resource created, returns 201 |
| Read resource | Returns expected data |
| Update resource | Changes persisted |
| Error case | Returns proper error response |

Note: IAM permissions are implicit - tests fail with AccessDeniedException if wrong.

### E2E Tests (API Gateway)
Test critical user journeys from external interface:

| Journey | Steps | Success Criteria |
|---------|-------|------------------|
| User registration | POST /register → verify email → login | User can authenticate |
| Password reset | Request reset → confirm → login with new password | Password changed |

### Test Environment
- Ephemeral stack per PR: `sam deploy -s feature-xyz`
- Cleanup after merge
```

### 10. Timeline

```markdown
## Timeline

### Milestones
| Milestone | Date | Deliverable |
|-----------|------|-------------|
| Design Complete | Jan 20 | Approved wireframes |
| API Complete | Jan 30 | Endpoints deployed to staging |
| Feature Complete | Feb 10 | Full flow working in staging |
| Launch | Feb 15 | Production release |

### Risks
| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Third-party email delays | Medium | High | Implement retry logic, fallback to SMS |
```

### 11. Open Questions

```markdown
## Open Questions

- [ ] Should we support phone number reset in v1?
- [ ] What's the token expiration policy?
- [x] Do we need CAPTCHA? → Yes, after 3 failed attempts
```

## Best Practices

### Be Specific
- Numbers over words ("under 2 seconds" not "fast")
- Examples over abstractions
- Screenshots over descriptions

### Stay Updated
- Mark status clearly
- Date all updates
- Track decision history

### Keep It Scannable
- Use tables for structured data
- Use bullet points for lists
- Bold key terms
- Include TL;DR for long sections

## Template

```markdown
# PRD: [Feature Name]

## Overview
**Owner:** @username | **Status:** Draft | **Updated:** YYYY-MM-DD

[One paragraph summary]

## Problem Statement
[What problem are we solving and for whom?]

## Goals & Success Metrics
| Metric | Current | Target |
|--------|---------|--------|
| | | |

## Scope
### In Scope
### Out of Scope

## User Stories
[Link to or include user stories]

## Requirements
### Functional
### Non-Functional

## Design
[Links to designs, key UX decisions]

## Technical Considerations
[Architecture notes, API changes, security]

## Test Plan
### Unit Tests (Mocked)
| Scenario | Input | Expected Response |
|----------|-------|-------------------|
| | | |

### Integration Tests (Real AWS)
| Scenario | What to Verify |
|----------|----------------|
| | |

### E2E Tests
| Journey | Steps | Success Criteria |
|---------|-------|------------------|
| | | |

## Open Questions
- [ ] Question 1
- [ ] Question 2
```
