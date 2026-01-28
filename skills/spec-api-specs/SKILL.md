---
name: spec-api-specs
description: API specification expert. Covers OpenAPI/Swagger, REST conventions, endpoints, request/response schemas, and error codes. Use for API design or spec questions.
model: opus
---

# API Specification Skill

Apply these patterns when designing API specifications.

## Additional Resources

- [Endpoints & Requests](./endpoints.md) - URL conventions, request/response, pagination
- [Error Handling](./errors.md) - Status codes, error format, error codes
- [Templates](./templates.md) - Complete OpenAPI template

## OpenAPI Structure

```yaml
openapi: 3.0.3
info:
  title: Service Name API
  version: 1.0.0
  description: Brief description of the API

servers:
  - url: https://api.example.com/v1
    description: Production
  - url: https://api.staging.example.com/v1
    description: Staging

paths:
  /resources:
    get:
      summary: List resources
      # ...

components:
  schemas:
    # Data models
  securitySchemes:
    # Authentication
```

## Authentication

### Bearer Token
```yaml
components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

security:
  - bearerAuth: []
```

### API Key
```yaml
components:
  securitySchemes:
    apiKey:
      type: apiKey
      in: header
      name: X-API-Key
```

## Versioning

### URL Versioning (Recommended)
```
https://api.example.com/v1/users
https://api.example.com/v2/users
```

### Header Versioning
```
Accept: application/vnd.api+json; version=1
```

## Best Practices

### Consistency
- Same patterns across all endpoints
- Consistent naming conventions
- Uniform error format

### Documentation
- Every endpoint has a summary
- All parameters documented
- Examples for request/response bodies
- Error cases documented

### Security
- Use HTTPS only
- Validate all input
- Return minimal data (no over-fetching)
- Implement rate limiting
