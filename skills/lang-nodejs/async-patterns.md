# Node.js Async Patterns

## Async/Await

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

## Error Handling

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

## Streams

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
