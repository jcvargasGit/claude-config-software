---
name: lang-golang
description: Go (Golang) language expert for *.go files. Covers Go version updates, idiomatic patterns, error handling, concurrency, testing, and code review. Use for any Go/Golang question.
model: opus
---

# Go Development Skill

Apply these Go-specific patterns and practices when working with Go code.

## Additional Resources

- [Patterns](./patterns.md) - Error handling, concurrency, interfaces
- [Architecture](./architecture.md) - Project structure (hexagonal), layer rules
- [Testing & Performance](./testing.md) - Table-driven tests, mocking, profiling

## Idiomatic Go

### Code Style
- Follow [Effective Go](https://go.dev/doc/effective_go) principles
- Use `gofmt` / `goimports` formatting
- Keep names short but descriptive (`i` for index, `r` for reader, `ctx` for context)
- Acronyms should be all caps (`HTTP`, `ID`, `URL`)
- Unexported names start lowercase, exported start uppercase
- More than 2 parameters: use structs (Go convention)

## Common Libraries

| Purpose | Library |
|---------|---------|
| Logging | `zerolog`, `zap`, `slog` (std) |
| HTTP Router | `chi`, `gin`, `echo` |
| Validation | `go-playground/validator` |
| Testing | `testify`, `gomock` |
| Config | `viper`, `envconfig` |
| AWS SDK | `aws-sdk-go-v2` |
| Database | `sqlx`, `pgx`, `gorm` |

## Lambda Patterns

```go
// Handler structure
func handler(ctx context.Context, event events.APIGatewayProxyRequest) (events.APIGatewayProxyResponse, error) {
    // Initialize outside handler for connection reuse
    // Parse request
    // Call business logic
    // Return response with proper status codes
}

// Cold start optimization
var (
    db     *dynamodb.Client
    once   sync.Once
    initErr error
)

func getDB() (*dynamodb.Client, error) {
    once.Do(func() {
        cfg, err := config.LoadDefaultConfig(context.Background())
        if err != nil {
            initErr = err
            return
        }
        db = dynamodb.NewFromConfig(cfg)
    })
    return db, initErr
}
```

## Quality Checklist

When writing Go code, verify:
- [ ] Errors are wrapped with context
- [ ] Context is propagated through call chain
- [ ] Resources are closed with defer
- [ ] Goroutines have proper cleanup (no leaks)
- [ ] Shared state is protected with mutex or channels
- [ ] Exported functions have doc comments
- [ ] Tests cover error paths
- [ ] No hardcoded configuration
