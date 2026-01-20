# API Error Handling

## HTTP Status Codes

### Success

| Code | Usage |
|------|-------|
| 200 | OK - General success |
| 201 | Created - Resource created |
| 204 | No Content - Success with no body (DELETE) |

### Client Errors

| Code | Usage |
|------|-------|
| 400 | Bad Request - Invalid input |
| 401 | Unauthorized - Authentication required |
| 403 | Forbidden - Insufficient permissions |
| 404 | Not Found - Resource doesn't exist |
| 409 | Conflict - Resource state conflict |
| 422 | Unprocessable Entity - Validation failed |
| 429 | Too Many Requests - Rate limited |

### Server Errors

| Code | Usage |
|------|-------|
| 500 | Internal Server Error |
| 502 | Bad Gateway |
| 503 | Service Unavailable |
| 504 | Gateway Timeout |

## Standard Error Format

```yaml
components:
  schemas:
    Error:
      type: object
      required:
        - code
        - message
      properties:
        code:
          type: string
          description: Machine-readable error code
          example: VALIDATION_ERROR
        message:
          type: string
          description: Human-readable message
          example: "Validation failed"
        details:
          type: array
          items:
            type: object
            properties:
              field:
                type: string
              message:
                type: string
          example:
            - field: email
              message: "Invalid email format"
            - field: password
              message: "Must be at least 8 characters"
```

## Error Codes

Define consistent error codes:

```yaml
# Error codes by category
AUTH_001: Invalid credentials
AUTH_002: Token expired
AUTH_003: Insufficient permissions

VALIDATION_001: Required field missing
VALIDATION_002: Invalid format
VALIDATION_003: Value out of range

RESOURCE_001: Not found
RESOURCE_002: Already exists
RESOURCE_003: State conflict
```
