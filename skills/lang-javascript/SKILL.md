---
name: lang-javascript
description: JavaScript expert for *.js/*.jsx files. Covers ES6+, async/await, modules, DOM, Node.js basics. Use for any JavaScript/JS question.
model: opus
---

# JavaScript Skill

Apply these JavaScript patterns and practices when working with JavaScript code.

## Code Style

- Prefer `const` over `let`, never use `var`
- Use strict equality (`===`)
- Prefer arrow functions for callbacks
- Use options object pattern for multiple parameters

## Modern Patterns

### Destructuring
```javascript
const { name, email } = user
const [first, second, ...rest] = items
const { data: userData = {} } = response

function createUser({ name, email, role = 'user' }) {
  return { name, email, role }
}
```

### Spread and Rest
```javascript
const merged = { ...defaults, ...options }
const copy = [...items]
const withNew = [...items, newItem]

function logAll(first, ...remaining) {
  console.log(first, remaining)
}
```

### Optional Chaining and Nullish Coalescing
```javascript
const city = user?.address?.city
const name = user.name ?? 'Anonymous'
const callback = options.onSuccess ?? (() => {})
```

### Template Literals
```javascript
const message = `Hello ${name}, you have ${count} notifications`
const multiline = `
  First line
  Second line
`
```

## Functions

### Arrow Functions
```javascript
const double = x => x * 2
const add = (a, b) => a + b
const process = data => {
  const result = transform(data)
  return result
}
```

### Default Parameters
```javascript
function fetchUsers(options = {}) {
  const { limit = 10, offset = 0 } = options
  return api.get('/users', { limit, offset })
}
```

### Higher-Order Functions
```javascript
const filtered = items.filter(item => item.active)
const mapped = items.map(item => item.name)
const total = items.reduce((sum, item) => sum + item.price, 0)
const found = items.find(item => item.id === targetId)
const exists = items.some(item => item.type === 'admin')
const allValid = items.every(item => item.valid)
```

## Async Patterns

### Async/Await
```javascript
async function fetchUser(id) {
  const response = await fetch(`/api/users/${id}`)
  if (!response.ok) return null
  return response.json()
}

async function fetchUserData(id) {
  const [user, orders] = await Promise.all([
    fetchUser(id),
    fetchOrders(id)
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
```

### Promise Utilities
```javascript
const results = await Promise.allSettled(promises)
const fulfilled = results
  .filter(r => r.status === 'fulfilled')
  .map(r => r.value)

const first = await Promise.race([fetchA(), fetchB()])
```

## Objects and Classes

### Object Methods
```javascript
const keys = Object.keys(user)
const values = Object.values(user)
const entries = Object.entries(user)
const merged = Object.assign({}, defaults, options)
const frozen = Object.freeze({ immutable: true })
```

### Classes
```javascript
class UserService {
  #apiClient

  constructor(apiClient) {
    this.#apiClient = apiClient
  }

  async findById(id) {
    return this.#apiClient.get(`/users/${id}`)
  }

  async create(userData) {
    return this.#apiClient.post('/users', userData)
  }
}
```

### Private Fields and Methods
```javascript
class Counter {
  #count = 0

  increment() {
    this.#count++
  }

  get value() {
    return this.#count
  }
}
```

## Arrays

### Common Operations
```javascript
const unique = [...new Set(items)]
const flat = nested.flat()
const flatMapped = items.flatMap(item => item.children)
const sorted = [...items].sort((a, b) => a.name.localeCompare(b.name))
const reversed = [...items].reverse()
const sliced = items.slice(0, 10)
```

### Array Creation
```javascript
const range = Array.from({ length: 10 }, (_, i) => i)
const filled = Array(5).fill(0)
const fromIterable = Array.from(nodeList)
```

## Modules

### Named Exports
```javascript
export function formatDate(date) {
  return date.toISOString()
}

export const DEFAULT_LIMIT = 10
```

### Default Export
```javascript
export default class ApiClient {
  // implementation
}
```

### Imports
```javascript
import ApiClient from './api-client'
import { formatDate, DEFAULT_LIMIT } from './utils'
import * as helpers from './helpers'
```

## Error Handling

### Custom Errors
```javascript
class ValidationError extends Error {
  constructor(field, message) {
    super(message)
    this.name = 'ValidationError'
    this.field = field
  }
}

function validate(data) {
  if (!data.email) {
    throw new ValidationError('email', 'Email is required')
  }
}
```

### Guard Clauses
```javascript
function processUser(user) {
  if (!user) return null
  if (!user.active) return null

  return {
    id: user.id,
    displayName: user.name.toUpperCase()
  }
}
```

## Quality Checklist

When writing JavaScript, verify:
- [ ] No `var` declarations
- [ ] Strict equality used (`===`)
- [ ] Async errors properly caught
- [ ] No unused variables
- [ ] Functions have single responsibility
- [ ] Arrays not mutated directly (use spread)
- [ ] No callback hell (use async/await)
- [ ] Meaningful variable names
