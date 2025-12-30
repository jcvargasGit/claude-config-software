---
name: lang-nodejs
description: Apply Node.js best practices, async patterns, and runtime-specific expertise when writing or reviewing Node.js code.
---

# Node.js Skill

Apply these Node.js patterns and practices when working with Node.js applications.

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

## Async Patterns

### Async/Await
```javascript
async function fetchUser(userId) {
  const response = await fetch(`/api/users/${userId}`)
  if (!response.ok) {
    throw new Error(`HTTP ${response.status}`)
  }
  return response.json()
}

async function fetchUserWithOrders(userId) {
  const [user, orders] = await Promise.all([
    fetchUser(userId),
    fetchOrders(userId)
  ])
  return { user, orders }
}
```

### Error Handling
```javascript
async function safeFetch(url) {
  try {
    const response = await fetch(url)
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`)
    }
    return { ok: true, data: await response.json() }
  } catch (error) {
    return { ok: false, error: error.message }
  }
}

// Handling multiple promises
async function processAll(items) {
  const results = await Promise.allSettled(
    items.map(item => processItem(item))
  )

  const successful = results
    .filter(r => r.status === 'fulfilled')
    .map(r => r.value)

  const failed = results
    .filter(r => r.status === 'rejected')
    .map(r => r.reason)

  return { successful, failed }
}
```

### Streams
```javascript
import { pipeline } from 'node:stream/promises'
import { createReadStream, createWriteStream } from 'node:fs'
import { createGzip } from 'node:zlib'

async function compressFile(input, output) {
  await pipeline(
    createReadStream(input),
    createGzip(),
    createWriteStream(output)
  )
}

// Async iteration over streams
import { createInterface } from 'node:readline'

async function processLines(filePath) {
  const rl = createInterface({
    input: createReadStream(filePath),
    crlfDelay: Infinity
  })

  for await (const line of rl) {
    await processLine(line)
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
  // Log to monitoring service
  process.exit(1)
})

// Uncaught exceptions
process.on('uncaughtException', (error) => {
  console.error('Uncaught Exception:', error)
  // Log to monitoring service
  process.exit(1)
})
```

## Lambda Handler Pattern

```javascript
export async function handler(event, context) {
  const requestId = context.awsRequestId

  try {
    const body = JSON.parse(event.body || '{}')
    const result = await processRequest(body)

    return {
      statusCode: 200,
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(result)
    }
  } catch (error) {
    console.error({ requestId, error: error.message, stack: error.stack })

    if (error instanceof ValidationError) {
      return {
        statusCode: 400,
        body: JSON.stringify({ error: error.message, code: error.code })
      }
    }

    if (error instanceof NotFoundError) {
      return {
        statusCode: 404,
        body: JSON.stringify({ error: error.message, code: error.code })
      }
    }

    return {
      statusCode: 500,
      body: JSON.stringify({ error: 'Internal server error' })
    }
  }
}
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

## Database Patterns

### Connection Pool
```javascript
import { DynamoDBClient } from '@aws-sdk/client-dynamodb'
import { DynamoDBDocumentClient } from '@aws-sdk/lib-dynamodb'

// Reuse connection across Lambda invocations
const client = new DynamoDBClient({})
export const docClient = DynamoDBDocumentClient.from(client, {
  marshallOptions: { removeUndefinedValues: true }
})
```

### Repository Pattern
```javascript
export class UserRepository {
  #tableName
  #client

  constructor(tableName, client) {
    this.#tableName = tableName
    this.#client = client
  }

  async findById(id) {
    const { Item } = await this.#client.send(
      new GetCommand({
        TableName: this.#tableName,
        Key: { pk: `USER#${id}`, sk: `USER#${id}` }
      })
    )
    return Item ?? null
  }

  async save(user) {
    await this.#client.send(
      new PutCommand({
        TableName: this.#tableName,
        Item: {
          pk: `USER#${user.id}`,
          sk: `USER#${user.id}`,
          ...user
        }
      })
    )
  }
}
```

## Testing

### Jest Setup
```javascript
// jest.config.js
export default {
  testEnvironment: 'node',
  transform: {},
  moduleNameMapper: {
    '^(\\.{1,2}/.*)\\.js$': '$1'
  },
  collectCoverageFrom: ['src/**/*.js'],
  coverageThreshold: {
    global: { branches: 80, functions: 80, lines: 80 }
  }
}
```

### Test Examples
```javascript
import { jest, describe, it, expect, beforeEach } from '@jest/globals'

describe('UserService', () => {
  let service
  let mockRepo

  beforeEach(() => {
    mockRepo = {
      findById: jest.fn(),
      save: jest.fn()
    }
    service = new UserService(mockRepo)
  })

  it('returns user when found', async () => {
    const user = { id: '123', email: 'test@example.com' }
    mockRepo.findById.mockResolvedValue(user)

    const result = await service.getUser('123')

    expect(result).toEqual(user)
    expect(mockRepo.findById).toHaveBeenCalledWith('123')
  })

  it('throws NotFoundError when user not found', async () => {
    mockRepo.findById.mockResolvedValue(null)

    await expect(service.getUser('invalid'))
      .rejects
      .toThrow(NotFoundError)
  })
})
```

## Project Structure

```
project/
├── src/
│   ├── index.js
│   ├── handlers/
│   │   ├── get-user.js
│   │   └── create-user.js
│   ├── domain/
│   │   ├── user.js
│   │   └── errors.js
│   ├── repository/
│   │   └── user-repository.js
│   └── lib/
│       ├── config.js
│       └── logger.js
├── tests/
│   ├── handlers/
│   └── domain/
├── package.json
└── jest.config.js
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
