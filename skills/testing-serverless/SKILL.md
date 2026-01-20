---
name: testing-serverless
description: Serverless testing expert using inverted test pyramid. Covers integration tests, Lambda testing, LocalStack, mocking AWS services, and contract testing. Use for testing Lambda or serverless apps.
model: opus
---

# Serverless Testing Skill

Apply these testing principles for serverless applications based on Yan Cui's testing strategy and AWS Lambda Testing Guide.

## The Serverless Test Honeycomb

The traditional test pyramid doesn't apply to serverless. Lambda functions are primarily I/O-heavy glue code, not complex business logic. Cloud-based tests provide the most accurate measure of quality.

### Inverted Pyramid for Serverless

| Level | Purpose | Distribution | Environment |
|-------|---------|--------------|-------------|
| Unit | Domain logic + failure simulation | 30% | Local, mocked |
| Integration | Cloud component flow | 50% | Ephemeral AWS |
| E2E | Critical user journeys | 20% | Deployed stack |

### Key Principle

> "Most tests should be integration tests against real AWS services" - Yan Cui

## Black-Box Testing Principle

**Test behavior, not implementation.** All tests should go through the handler (entry point), treating the system as a black box:

- Use HTTP inputs/outputs (what clients see)
- Don't reference internal types (`NewEmail`, `ErrInvalidEmail`)
- Test the API contract, not internal functions

### Test Format

| Scenario | Input | Expected Response |
|----------|-------|-------------------|
| Successful registration | POST /auth/register with valid email/password | 201 + confirmation message |
| Invalid email format | POST /auth/register with email: "invalid" | 400 + "Invalid email" error |
| Service unavailable | POST /auth/register (mocked service down) | 503 + "Service unavailable" |

## Unit Tests - When & What

### When to Write Unit Tests

- **Failure scenario simulation** (primary use case - hard to trigger in cloud)
- Input validation edge cases
- Error response formatting

### What to Mock (Simulate Failures)

Mocks are most valuable for simulating failures via the handler:

- Service unavailable / network errors → 503 response
- Rate limiting (TooManyRequests) → 429 response
- Invalid credentials / expired tokens → 401 response
- Resource not found → 404 response
- Timeout scenarios → 504 response

### What NOT to Unit Test

- Happy paths (test in cloud instead)
- AWS SDK calls directly (mock APIs drift from reality)
- IAM permissions (implicit - tests fail if wrong)
- Internal domain types directly

## Integration Tests - The Foundation

### Remocal Testing

"Remocal" = Run locally against remote AWS services

- Deploy ephemeral stack: `sam deploy -s feature-xyz`
- Execute tests locally against real DynamoDB, Cognito, S3
- Fast feedback loop with high confidence
- Catches IAM, configuration, and integration issues

### What to Test in Integration

- Actual service interactions (happy paths)
- Event source mappings (SQS -> Lambda)
- Real AWS SDK behavior
- Data format compatibility
- Error responses from real services

Note: IAM permissions are tested implicitly - if wrong, tests fail with AccessDeniedException.

### Integration Test Benefits

- Tests every available AWS service
- Always uses latest service APIs
- Matches production environment closely
- Catches configuration issues early

## E2E Tests - External Interfaces

### Test from External-Facing Interfaces

- REST API endpoints via API Gateway
- EventBridge event flows
- Step Functions workflows
- WebSocket connections

### Reuse Test Cases

- Same scenarios as integration tests
- Different entry point (API vs direct invocation)
- Validates full request/response cycle

## Ephemeral Environments Strategy

### Developer Workflow

1. Create feature branch
2. Deploy ephemeral stack: `sam deploy -s my-feature`
3. Run integration tests locally against cloud
4. Only redeploy when infrastructure changes
5. Destroy when done: `sam remove -s my-feature`

### CI/CD Pipeline

- Create ephemeral environment per PR
- Run full test suite against real services
- Destroy environment after merge
- Prevents test data pollution

### Cost Management

- Set up billing alerts per environment
- Use resource tagging for cost tracking
- Auto-delete stacks after 24h of inactivity

## Mocking Strategy for Serverless

### Use Mocks ONLY For

- Simulating failure conditions (hard to trigger in cloud)
- Complex business logic with many edge cases
- Offline development (temporary measure)
- Unit testing domain layer

### NEVER Rely on Mocks For

- Validating AWS service calls work
- Testing IAM permissions
- Verifying event source mappings
- Configuration validation
- Service quota testing

### Mock Limitations

| Issue | Impact |
|-------|--------|
| APIs drift from actual AWS APIs | False positives |
| Tests pass locally, fail in cloud | Wasted debugging time |
| Cannot test IAM, quotas, configs | Production failures |
| Incomplete service behavior | Missing edge cases |

## Code Structure for Testability

### Hexagonal Architecture Benefits

- Domain logic isolated from I/O
- Ports (interfaces) enable mocking for failure tests
- Adapters tested via integration tests
- Clear separation of concerns

### Lambda Handler Pattern

```
handler(event) ->
  1. Parse and validate event (input adapter)
  2. Call use case (application layer)
  3. Format response (output adapter)
```

### Test by Scenario Type

| Scenario Type | Test Level | Approach |
|---------------|------------|----------|
| Validation errors | Unit (mocked) | POST with invalid data → 400 |
| Service failures | Unit (mocked) | Mock dependency down → 503 |
| Happy paths | Integration | Real AWS services |
| User journeys | E2E | Full API Gateway request/response |

## What Cloud Tests Catch That Mocks Miss

| Scenario | Mock Result | Cloud Reality |
|----------|-------------|---------------|
| Lambda creates S3 bucket | Pass | IAM role lacks s3:CreateBucket |
| SQS triggers Lambda | Pass | Visibility timeout too low |
| Lambda writes logs | Pass | Missing logs:PutLogEvents permission |
| Lambda timeout | Pass | Configured timeout too short |
| DynamoDB query | Pass | Missing GSI, wrong partition key |
| Cognito auth | Pass | App client misconfigured |

## Test Organization

### By Feature, Not by Class

```
tests/
  auth/
    register_test.go       # All register scenarios
    login_test.go          # All login scenarios
  integration/
    cognito_test.go        # Real Cognito interactions
  e2e/
    auth_api_test.go       # API Gateway endpoints
```

### Naming Convention

Use descriptive names that read like documentation:

- `TestRegister_WithValidEmail_ReturnsSuccess`
- `TestRegister_WithExistingEmail_ReturnsUserExistsError`
- `TestLogin_WhenUserNotVerified_ReturnsNotVerifiedError`

## Quality Checklist

When designing tests for serverless applications:

- [ ] Tests use black-box approach (HTTP inputs/outputs)
- [ ] Unit tests cover failure scenarios using mocks
- [ ] Integration tests run against real AWS services (happy paths)
- [ ] Ephemeral environment created for feature/PR
- [ ] Event source mappings tested end-to-end
- [ ] No internal types referenced in test scenarios
- [ ] No LocalStack dependency for critical paths
- [ ] Test data cleanup after integration tests

## Anti-Patterns to Avoid

### Over-Mocking

- Don't mock AWS SDK for happy path tests
- Don't trust mocks for permission validation
- Don't use LocalStack as primary test strategy

### Under-Testing in Cloud

- Don't skip integration tests to save time
- Don't assume local tests guarantee cloud success
- Don't ignore IAM and configuration testing

### Test Isolation Issues

- Don't share state between tests
- Don't depend on specific resource names
- Don't leave test data in shared environments

## References

- [Yan Cui - Testing Strategy for Serverless](https://theburningmonk.com/2022/05/my-testing-strategy-for-serverless-applications/)
- [AWS Lambda Testing Guide](https://docs.aws.amazon.com/lambda/latest/dg/testing-guide.html)
- [Serverless Test Samples](https://github.com/aws-samples/serverless-test-samples)
