---
name: spec-user-stories
description: User story expert. Covers "As a... I want... So that..." format, INVEST criteria, story points, and backlog refinement. Use for writing user stories or tickets.
model: opus
---

# User Stories Skill

Apply these patterns when writing user stories.

## Format

### Standard Template
```
As a [user type],
I want [action/capability],
So that [benefit/value].
```

### Examples

**Good:**
```
As a registered customer,
I want to save items to a wishlist,
So that I can purchase them later without searching again.
```

**Bad:**
```
As a user,
I want a wishlist feature,
So that I can use it.
```

## Components

### User Type (Who)
- Be specific about the persona
- Distinguish between user roles when behavior differs
- Examples: "first-time visitor", "premium subscriber", "account admin"

### Action (What)
- Focus on the user's goal, not the solution
- Use active verbs
- Be specific enough to be testable

### Benefit (Why)
- Explain the business or user value
- Connect to measurable outcomes
- Avoid circular reasoning ("so that I have the feature")

## Story Sizing

### Epic
Large body of work that can be broken into smaller stories.
```
Epic: User Authentication
├── Story: Email/password registration
├── Story: Social login (Google, GitHub)
├── Story: Password reset
└── Story: Two-factor authentication
```

### Story
Deliverable in a single sprint, provides user value.

### Task
Technical work within a story (not user-facing).

## INVEST Criteria

Good user stories are:

| Criteria | Description |
|----------|-------------|
| **I**ndependent | Can be developed in any order |
| **N**egotiable | Details can be discussed |
| **V**aluable | Delivers user or business value |
| **E**stimable | Team can estimate effort |
| **S**mall | Fits in a sprint |
| **T**estable | Clear pass/fail criteria |

## Story Mapping

### Horizontal: User Journey
```
Discovery → Registration → Onboarding → Core Usage → Retention
```

### Vertical: Priority
```
Must Have (MVP)
Should Have
Could Have
Won't Have (this release)
```

## Edge Cases

Always consider and document:
- First-time vs returning users
- Empty states
- Error states
- Permission boundaries
- Rate limits / quotas

## Anti-Patterns

### Avoid

| Pattern | Problem |
|---------|---------|
| Technical stories | "As a developer, I want to refactor..." |
| Solution-first | "As a user, I want a dropdown menu..." |
| Compound stories | Multiple independent features in one |
| Vague benefit | "So that it works better" |

### Prefer

| Instead of | Write |
|------------|-------|
| "As a user..." | "As a [specific role]..." |
| "I want feature X" | "I want to [achieve goal]" |
| "So that I can use it" | "So that [measurable outcome]" |

## Template

When writing user stories, include:

```markdown
## [Story Title]

As a [user type],
I want [action],
So that [benefit].

### Acceptance Criteria
- [ ] Given... When... Then...

### Notes
- Assumptions
- Dependencies
- Out of scope

### Open Questions
- [ ] Question needing answer
```
