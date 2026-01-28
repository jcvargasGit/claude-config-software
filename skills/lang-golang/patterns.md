# Go Patterns

## Error Handling

```go
// Wrap errors with context (Go 1.13+)
if err != nil {
    return fmt.Errorf("failed to process order %s: %w", orderID, err)
}

// Check for specific errors
if errors.Is(err, sql.ErrNoRows) {
    return nil, ErrNotFound
}

// Type assert errors
var validationErr *ValidationError
if errors.As(err, &validationErr) {
    // handle validation error
}
```

## Concurrency Patterns

```go
// Always pass context for cancellation
func DoWork(ctx context.Context) error {
    select {
    case <-ctx.Done():
        return ctx.Err()
    case result := <-workChan:
        return process(result)
    }
}

// Use errgroup for concurrent operations
g, ctx := errgroup.WithContext(ctx)
for _, item := range items {
    item := item // capture loop variable
    g.Go(func() error {
        return processItem(ctx, item)
    })
}
if err := g.Wait(); err != nil {
    return err
}

// Protect shared state with mutex
type Counter struct {
    mu    sync.Mutex
    count int
}

func (c *Counter) Inc() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
}
```

## Interface Design

```go
// Define small interfaces at point of use
type Reader interface {
    Read(p []byte) (n int, err error)
}

// Accept interfaces, return structs
func NewService(repo Repository) *Service {
    return &Service{repo: repo}
}

// Use interface{} sparingly, prefer generics in Go 1.18+
func Map[T, U any](items []T, fn func(T) U) []U {
    result := make([]U, len(items))
    for i, item := range items {
        result[i] = fn(item)
    }
    return result
}
```
