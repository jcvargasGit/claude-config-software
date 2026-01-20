---
name: spec-acceptance-criteria
description: Acceptance criteria expert using BDD Given/When/Then format. Covers testable requirements, edge cases, and definition of done. Use for writing AC or acceptance criteria.
model: opus
---

# Acceptance Criteria Skill

Apply these patterns when writing acceptance criteria.

## Format

### Given/When/Then (Gherkin)

```gherkin
Given [precondition/context]
When [action/trigger]
Then [expected outcome]
```

### Examples

**Good:**
```gherkin
Given a user is logged in
And has items in their cart
When they click "Checkout"
Then they are redirected to the payment page
And the cart total is displayed
```

**Bad:**
```
- User can checkout
- Payment works
```

## Components

### Given (Context)
- System state before action
- User state/role
- Data preconditions
- Can chain with "And"

```gherkin
Given the user has a verified email
And has not exceeded their monthly limit
And the feature flag "new-checkout" is enabled
```

### When (Action)
- Single user action or system event
- Be specific about the trigger
- One "When" per scenario (usually)

```gherkin
When the user submits the registration form
When the payment webhook is received
When 24 hours pass since last login
```

### Then (Outcome)
- Observable result
- Can include multiple assertions
- Must be testable

```gherkin
Then a confirmation email is sent
And the user's status changes to "verified"
And an audit log entry is created
```

## Scenario Types

### Happy Path
The expected, successful flow.
```gherkin
Scenario: Successful password reset
Given a user with email "user@example.com" exists
When they request a password reset
Then they receive a reset email within 5 minutes
```

### Edge Cases
Boundary conditions and limits.
```gherkin
Scenario: Password at minimum length
Given the minimum password length is 8 characters
When a user enters an 8-character password
Then the password is accepted

Scenario: Password below minimum length
Given the minimum password length is 8 characters
When a user enters a 7-character password
Then the error "Password must be at least 8 characters" is shown
```

### Error Handling
What happens when things go wrong.
```gherkin
Scenario: Payment declined
Given a user is on the payment page
When they submit a card that is declined
Then the error "Payment declined. Please try another card." is shown
And no order is created
And the cart is preserved
```

### Security
Permission and access control.
```gherkin
Scenario: Non-admin cannot access admin panel
Given a user with role "member"
When they navigate to /admin
Then they are redirected to /dashboard
And a 403 status is logged
```

## Writing Guidelines

### Be Specific

| Vague | Specific |
|-------|----------|
| User sees error | User sees "Email already registered" error |
| Page loads fast | Page loads in under 2 seconds |
| Email is sent | Email with subject "Reset your password" is sent within 5 minutes |

### Be Testable

Every criterion should answer:
- What is the input/action?
- What is the expected output?
- How do we verify it?

### Include Data

```gherkin
Given the following users exist:
| email            | role    | status   |
| admin@test.com   | admin   | active   |
| user@test.com    | member  | active   |
| banned@test.com  | member  | banned   |
```

## Common Patterns

### State Transitions
```gherkin
Given an order with status "pending"
When payment is confirmed
Then the order status changes to "processing"
And the customer receives an order confirmation email
```

### Time-Based
```gherkin
Given a password reset token was created 2 hours ago
And tokens expire after 1 hour
When the user clicks the reset link
Then the error "This link has expired" is shown
```

### Concurrency
```gherkin
Given only 1 ticket remains for the event
When two users click "Purchase" simultaneously
Then only one user completes the purchase
And the other sees "Sold out"
```

## Anti-Patterns

### Avoid

| Pattern | Problem |
|---------|---------|
| Implementation details | "When the API returns 200..." |
| Multiple actions | "When user logs in and goes to settings and changes password..." |
| Ambiguous outcomes | "Then everything works correctly" |
| UI-specific | "When user clicks the blue button..." |

## Template

```markdown
## Feature: [Name]

### Scenario: [Happy path description]
Given [context]
When [action]
Then [outcome]

### Scenario: [Edge case description]
Given [context]
When [action]
Then [outcome]

### Scenario: [Error case description]
Given [context]
When [action]
Then [error handling]
```
