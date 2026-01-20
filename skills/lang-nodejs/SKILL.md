---
name: lang-nodejs
description: Node.js runtime expert. Covers npm, package.json, async patterns, streams, Express, debugging, and performance. Use for any Node/npm question.
model: opus
---

# Node.js Skill

Apply these Node.js patterns and practices when working with Node.js applications.

## Additional Resources

- [Async Patterns](./async-patterns.md) - Async/await, streams, error handling
- [Testing](./testing.md) - Jest setup, mocking, examples
- [Examples](./examples.md) - Lambda handler, repository pattern, project structure

## Code Style

- Use ES modules (`import`/`export`)
- Prefer `const` over `let`, never `var`
- Use async/await over callbacks
- Handle all promise rejections
- Use strict equality (`===`)

## Module System

### ES Modules
```javascript
// Named exports
export function processOrder(order) {
  return validate(order)
}

export const DEFAULT_TIMEOUT = 5000

// Default export
export default class OrderService {
  // implementation
}

// Imports
import OrderService from './order-service.js'
import { processOrder, DEFAULT_TIMEOUT } from './utils.js'
import * as helpers from './helpers.js'
```

### Package.json ES Modules
```json
{
  "name": "my-service",
  "type": "module",
  "engines": {
    "node": ">=20.0.0"
  },
  "exports": {
    ".": "./src/index.js",
    "./utils": "./src/utils.js"
  }
}
```

## Error Handling

### Custom Errors
```javascript
class AppError extends Error {
  constructor(message, code, statusCode = 500) {
    super(message)
    this.name = this.constructor.name
    this.code = code
    this.statusCode = statusCode
    Error.captureStackTrace(this, this.constructor)
  }
}

class NotFoundError extends AppError {
  constructor(resource, id) {
    super(`${resource} not found: ${id}`, 'NOT_FOUND', 404)
    this.resource = resource
    this.id = id
  }
}

class ValidationError extends AppError {
  constructor(field, message) {
    super(`${field}: ${message}`, 'VALIDATION_ERROR', 400)
    this.field = field
  }
}
```

### Global Error Handling
```javascript
// Unhandled rejections
process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection:', reason)
  process.exit(1)
})

// Uncaught exceptions
process.on('uncaughtException', (error) => {
  console.error('Uncaught Exception:', error)
  process.exit(1)
})
```

## Environment Configuration

```javascript
import { z } from 'zod'

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'staging', 'production']).default('development'),
  PORT: z.coerce.number().default(3000),
  DATABASE_URL: z.string().url(),
  API_KEY: z.string().min(1),
  LOG_LEVEL: z.enum(['debug', 'info', 'warn', 'error']).default('info')
})

export const config = envSchema.parse(process.env)
```

## Logging

```javascript
import pino from 'pino'

export const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  formatters: {
    level: (label) => ({ level: label })
  }
})

// Usage
logger.info({ userId, action: 'login' }, 'User logged in')
logger.error({ err, requestId }, 'Request failed')
```

## Quality Checklist

When writing Node.js code, verify:
- [ ] All promises have error handling
- [ ] No callback-based APIs (use promisify or native promises)
- [ ] Environment variables validated at startup
- [ ] Connections reused across Lambda invocations
- [ ] Structured logging with context
- [ ] Graceful shutdown handling
- [ ] No synchronous file operations in hot paths
- [ ] Dependencies audited for vulnerabilities
