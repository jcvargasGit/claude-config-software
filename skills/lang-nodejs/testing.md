# Node.js Testing

## Jest Setup

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

## Test Examples

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
