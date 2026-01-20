# Go Testing & Performance

## Table-Driven Tests

```go
func TestAdd(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive numbers", 2, 3, 5},
        {"negative numbers", -1, -2, -3},
        {"zero", 0, 0, 0},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            result := Add(tt.a, tt.b)
            if result != tt.expected {
                t.Errorf("Add(%d, %d) = %d; want %d", tt.a, tt.b, result, tt.expected)
            }
        })
    }
}
```

## Using Testify

```go
func TestService(t *testing.T) {
    assert := assert.New(t)
    require := require.New(t)

    result, err := service.Process(input)
    require.NoError(err)
    assert.Equal(expected, result)
}
```

## Mocking

```go
// Mock interfaces with gomock or manual mocks
type mockRepository struct {
    items map[string]Item
}

func (m *mockRepository) Get(id string) (Item, error) {
    item, ok := m.items[id]
    if !ok {
        return Item{}, ErrNotFound
    }
    return item, nil
}
```

## Memory Optimization

```go
// Pre-allocate slices when size is known
items := make([]Item, 0, len(input))

// Use sync.Pool for frequently allocated objects
var bufferPool = sync.Pool{
    New: func() interface{} {
        return new(bytes.Buffer)
    },
}

buf := bufferPool.Get().(*bytes.Buffer)
defer bufferPool.Put(buf)
buf.Reset()

// Avoid string concatenation in loops
var sb strings.Builder
for _, s := range items {
    sb.WriteString(s)
}
result := sb.String()
```

## Profiling

```bash
# CPU profiling
go test -cpuprofile=cpu.prof -bench=.
go tool pprof cpu.prof

# Memory profiling
go test -memprofile=mem.prof -bench=.
go tool pprof mem.prof
```

## Benchmarking

```go
func BenchmarkProcess(b *testing.B) {
    for i := 0; i < b.N; i++ {
        Process(input)
    }
}
```
