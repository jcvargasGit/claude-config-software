# Node.js Examples

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
