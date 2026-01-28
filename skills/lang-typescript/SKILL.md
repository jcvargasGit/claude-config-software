---
name: lang-typescript
description: TypeScript expert for *.ts/*.tsx files. Covers types, generics, interfaces, React+TS, Node+TS, and tsconfig. Use for any TypeScript question.
model: opus
---

# TypeScript Skill

Apply these TypeScript patterns and practices when working with TypeScript code.

## Code Style

- Prefer `const` over `let`, never use `var`
- Use strict mode (`"strict": true` in tsconfig)
- Prefer explicit return types for public functions
- Use options object pattern for multiple parameters

## Type Patterns

### Prefer Interfaces for Objects
```typescript
interface User {
  id: string
  email: string
  createdAt: Date
}

interface CreateUserInput {
  email: string
  password: string
}
```

### Use Type for Unions and Utilities
```typescript
type Status = 'pending' | 'active' | 'inactive'
type UserWithRole = User & { role: Role }
type PartialUser = Partial<User>
type UserKeys = keyof User
```

### Discriminated Unions
```typescript
type Result<T> =
  | { success: true; data: T }
  | { success: false; error: string }

function handleResult<T>(result: Result<T>) {
  if (result.success) {
    console.log(result.data)
  } else {
    console.error(result.error)
  }
}
```

### Generic Constraints
```typescript
interface HasId {
  id: string
}

function findById<T extends HasId>(items: T[], id: string): T | undefined {
  return items.find(item => item.id === id)
}
```

## Function Patterns

### Options Object Pattern
```typescript
interface FetchUsersOptions {
  limit?: number
  offset?: number
  sortBy?: keyof User
}

async function fetchUsers(options: FetchUsersOptions = {}): Promise<User[]> {
  const { limit = 10, offset = 0, sortBy = 'createdAt' } = options
  // implementation
}
```

### Result Types Over Exceptions
```typescript
type ApiResult<T> =
  | { ok: true; data: T }
  | { ok: false; error: ApiError }

async function createUser(input: CreateUserInput): Promise<ApiResult<User>> {
  try {
    const user = await api.post('/users', input)
    return { ok: true, data: user }
  } catch (e) {
    return { ok: false, error: parseError(e) }
  }
}
```

### Type Guards
```typescript
function isUser(value: unknown): value is User {
  return (
    typeof value === 'object' &&
    value !== null &&
    'id' in value &&
    'email' in value
  )
}
```

## Utility Types

```typescript
// Pick specific properties
type UserPreview = Pick<User, 'id' | 'email'>

// Omit properties
type UserWithoutId = Omit<User, 'id'>

// Make all optional
type PartialUser = Partial<User>

// Make all required
type RequiredUser = Required<User>

// Make readonly
type ReadonlyUser = Readonly<User>

// Record for dictionaries
type UserMap = Record<string, User>

// Extract from union
type ActiveStatus = Extract<Status, 'active'>

// Exclude from union
type InactiveStatuses = Exclude<Status, 'active'>
```

## Async Patterns

```typescript
// Async function with proper typing
async function fetchUser(id: string): Promise<User | null> {
  const response = await fetch(`/api/users/${id}`)
  if (!response.ok) return null
  return response.json()
}

// Promise.all with tuple types
async function fetchUserData(id: string) {
  const [user, orders] = await Promise.all([
    fetchUser(id),
    fetchOrders(id)
  ])
  return { user, orders }
}

// Handle multiple async operations
async function processUsers(ids: string[]): Promise<User[]> {
  const results = await Promise.allSettled(ids.map(fetchUser))
  return results
    .filter((r): r is PromiseFulfilledResult<User> => r.status === 'fulfilled')
    .map(r => r.value)
}
```

## Module Patterns

### Barrel Exports
```typescript
// domain/index.ts
export { User, CreateUserInput } from './user'
export { Order, OrderStatus } from './order'
export * from './errors'
```

### Namespace Imports
```typescript
import * as UserService from './services/user'

UserService.create(input)
UserService.findById(id)
```

## Strict Null Handling

```typescript
// Optional chaining
const city = user?.address?.city

// Nullish coalescing
const name = user.name ?? 'Anonymous'

// Non-null assertion (use sparingly)
const element = document.getElementById('app')!

// Type narrowing
function processUser(user: User | null) {
  if (!user) return
  // user is now User, not User | null
  console.log(user.email)
}
```

## Configuration

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  }
}
```

## Testing

### Test Style by Level

| Level | Style | TypeScript Pattern |
|-------|-------|-------------------|
| **Unit** | Simple, describe/it | Direct assertions |
| **Integration/E2E** | BDD Given/When/Then | Step objects |

---

### Unit Tests: Simple (describe/it)

### Jest/Vitest Setup

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    coverage: { reporter: ['text', 'lcov'] }
  }
})
```

### Basic Tests

```typescript
import { describe, it, expect, beforeAll, afterAll } from 'vitest'

describe('UserService', () => {
  beforeAll(async () => {
    // Setup before all tests
  })

  afterAll(async () => {
    // Cleanup after all tests
  })

  it('should create a user', async () => {
    const result = await createUser({ email: 'test@example.com' })
    expect(result.ok).toBe(true)
    if (result.ok) {
      expect(result.data.email).toBe('test@example.com')
    }
  })
})
```

---

### Integration/E2E Tests: BDD Step Pattern

```typescript
// steps/given.ts
export const given = {
  aUserExists: async (t: TestContext): Promise<User> => {
    const user = await createTestUser()
    t.onTestFinished(() => deleteTestUser(user.id))
    return user
  }
}

// steps/when.ts
export const when = {
  callLogin: async (email: string, password: string): Promise<Response> => {
    return fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify({ email, password })
    })
  }
}

// steps/then.ts
export const then = {
  statusCodeIs: (response: Response, expected: number) => {
    expect(response.status).toBe(expected)
  },
  responseContainsToken: async (response: Response) => {
    const body = await response.json()
    expect(body.accessToken).toBeDefined()
  }
}

// feature.test.ts
describe('Login', () => {
  it('should login successfully', async (t) => {
    // Given
    const user = await given.aUserExists(t)

    // When
    const response = await when.callLogin(user.email, 'password')

    // Then
    then.statusCodeIs(response, 200)
    await then.responseContainsToken(response)
  })
})
```

### Mocking

```typescript
import { vi, Mock } from 'vitest'

// Mock a module
vi.mock('./api', () => ({
  fetchUser: vi.fn()
}))

// Type-safe mock
const mockFetchUser = fetchUser as Mock
mockFetchUser.mockResolvedValue({ id: '1', email: 'test@example.com' })
```

## Observability

### Structured Logging with Pino

```typescript
import pino from 'pino'

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  base: {
    service: 'my-service',
    stage: process.env.STAGE
  }
})

// Info level
logger.info({ method: 'POST', path: '/users', status: 200, duration: 42 }, 'request handled')

// Warn for 4xx
logger.warn({ err, operation: 'login', status: 401 }, 'request failed')

// Error for 5xx
logger.error({ err, operation: 'createUser', status: 500 }, 'request failed')
```

### Error Logging Helper

```typescript
function logError(err: Error, operation: string): ErrorResponse {
  const status = mapErrorToStatus(err)
  const level = status >= 500 ? 'error' : 'warn'

  logger[level]({ err, operation, status }, 'request failed')

  return errorResponse(err)
}

// Usage
if (!result.ok) {
  return logError(result.error, 'register')
}
```

### Request Context Middleware

```typescript
import { AsyncLocalStorage } from 'async_hooks'

interface RequestContext {
  requestId: string
  logger: pino.Logger
}

const storage = new AsyncLocalStorage<RequestContext>()

function requestMiddleware(req: Request, res: Response, next: NextFunction) {
  const requestId = req.headers['x-request-id'] || crypto.randomUUID()
  const childLogger = logger.child({ requestId })

  storage.run({ requestId, childLogger }, () => next())
}

// Get logger in any function
function getLogger(): pino.Logger {
  return storage.getStore()?.logger || logger
}
```

## Quality Checklist

When writing TypeScript code, verify:
- [ ] No `any` types (use `unknown` if needed)
- [ ] Strict null checks handled
- [ ] Public functions have explicit return types
- [ ] Complex objects use interfaces
- [ ] Error cases return Result types or throw typed errors
- [ ] Generics used where appropriate
- [ ] No type assertions without validation
- [ ] Async errors properly caught
