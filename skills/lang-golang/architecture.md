# Go Project Architecture (Hexagonal)

Go implementation of hexagonal architecture. See `arch-hexagonal` skill for concepts.

## Project Structure

```
app/
├── cmd/
│   ├── api/
│   │   └── main.go           # HTTP API entrypoint
│   └── worker/
│       └── main.go           # Event worker entrypoint
├── internal/
│   ├── domain/               # Core business logic (no dependencies)
│   │   ├── user/
│   │   │   ├── entity.go     # User entity
│   │   │   ├── repository.go # Repository interface (output port)
│   │   │   └── service.go    # Domain service
│   │   └── order/
│   │       ├── entity.go
│   │       ├── repository.go
│   │       └── service.go
│   ├── application/          # Use cases / Application services
│   │   ├── dto/              # Input/Output DTOs
│   │   │   └── user.go
│   │   └── usecase/          # One file per use case
│   │       ├── create_user.go
│   │       └── get_user.go
│   ├── adapter/
│   │   ├── input/            # Driving adapters
│   │   │   ├── http/         # HTTP handlers
│   │   │   │   ├── handler.go
│   │   │   │   ├── router.go
│   │   │   │   └── middleware.go
│   │   │   └── lambda/       # Lambda handlers
│   │   │       └── handler.go
│   │   └── output/           # Driven adapters
│   │       ├── persistence/  # Database implementations
│   │       │   ├── dynamodb/
│   │       │   │   └── user_repository.go
│   │       │   └── postgres/
│   │       │       └── user_repository.go
│   │       └── gateway/      # External API clients
│   │           └── payment/
│   │               └── stripe.go
│   └── config/               # Configuration loading
│       └── config.go
├── pkg/                      # Shared public libraries (if any)
├── go.mod
├── go.sum
└── Makefile
```

## Layer Rules

```go
// domain/user/repository.go - Output port (interface in domain)
type Repository interface {
    FindByID(ctx context.Context, id string) (*User, error)
    Save(ctx context.Context, user *User) error
}

// domain/user/entity.go - Pure domain, no external imports
type User struct {
    ID        string
    Email     string
    CreatedAt time.Time
}

func NewUser(email string) (*User, error) {
    if email == "" {
        return nil, ErrInvalidEmail
    }
    return &User{
        ID:        uuid.NewString(),
        Email:     email,
        CreatedAt: time.Now(),
    }, nil
}

// application/usecase/create_user.go - Use case with injected port
type CreateUserUseCase struct {
    repo user.Repository  // depends on interface, not implementation
}

func (uc *CreateUserUseCase) Execute(ctx context.Context, input dto.CreateUserInput) (*dto.UserOutput, error) {
    u, err := user.NewUser(input.Email)
    if err != nil {
        return nil, err
    }
    if err := uc.repo.Save(ctx, u); err != nil {
        return nil, err
    }
    return &dto.UserOutput{ID: u.ID, Email: u.Email}, nil
}

// adapter/output/persistence/dynamodb/user_repository.go - Driven adapter
type UserRepository struct {
    client *dynamodb.Client
    table  string
}

func (r *UserRepository) FindByID(ctx context.Context, id string) (*user.User, error) {
    // Implementation using DynamoDB client
}

// cmd/api/main.go - Wire everything together
func main() {
    // 1. Create driven adapters
    userRepo := dynamodb.NewUserRepository(client, tableName)

    // 2. Create use cases with output ports
    createUser := usecase.NewCreateUserUseCase(userRepo)

    // 3. Create driving adapters with use cases
    handler := http.NewHandler(createUser)

    // 4. Start
    router := http.NewRouter(handler)
    router.ListenAndServe(":8080")
}
```
