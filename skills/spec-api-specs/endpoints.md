# API Endpoints & Requests

## URL Conventions

```
GET    /users          # List users
POST   /users          # Create user
GET    /users/{id}     # Get user
PUT    /users/{id}     # Replace user
PATCH  /users/{id}     # Update user fields
DELETE /users/{id}     # Delete user

GET    /users/{id}/orders    # List user's orders (nested resource)
```

### Naming Rules
- Use plural nouns for collections (`/users` not `/user`)
- Use kebab-case for multi-word (`/user-profiles`)
- Use path parameters for identifiers (`/users/{userId}`)
- Use query parameters for filtering (`/users?status=active`)

## Request Body

```yaml
requestBody:
  required: true
  content:
    application/json:
      schema:
        type: object
        required:
          - email
          - password
        properties:
          email:
            type: string
            format: email
            example: user@example.com
          password:
            type: string
            minLength: 8
            example: "securePassword123"
          name:
            type: string
            maxLength: 100
```

## Response Body

```yaml
responses:
  '200':
    description: Successful response
    content:
      application/json:
        schema:
          type: object
          properties:
            id:
              type: string
              format: uuid
            email:
              type: string
            createdAt:
              type: string
              format: date-time
```

## Pagination

```yaml
# Request
GET /users?page=2&limit=20

# Response
{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 150,
    "totalPages": 8
  }
}
```

## Filtering & Sorting

```yaml
# Filtering
GET /orders?status=pending&createdAfter=2025-01-01

# Sorting
GET /products?sort=price:asc,name:desc
```

## Rate Limiting

Document rate limits in responses:

```yaml
headers:
  X-RateLimit-Limit:
    description: Requests allowed per window
    schema:
      type: integer
  X-RateLimit-Remaining:
    description: Requests remaining in window
    schema:
      type: integer
  X-RateLimit-Reset:
    description: Unix timestamp when window resets
    schema:
      type: integer
```
