---
name: arch-hexagonal
description: Hexagonal Architecture (Ports & Adapters) expert. Covers domain-driven design, clean architecture, dependency inversion, and testable code structure. Use for architecture questions.
model: opus
---

# Hexagonal Architecture Skill

Apply these Hexagonal Architecture (Ports & Adapters) patterns when designing applications. This is language-agnostic.

## Core Concept

Hexagonal Architecture isolates business logic from external concerns (databases, APIs, frameworks) through **ports** (interfaces) and **adapters** (implementations).

```
                    ┌─────────────────────────────────────┐
                    │         INPUT ADAPTERS              │
                    │  HTTP Handler, CLI, gRPC, Events    │
                    └──────────────┬──────────────────────┘
                                   │ calls
                                   ▼
                    ┌─────────────────────────────────────┐
                    │         INPUT PORTS (Interfaces)    │
                    │         UseCase / Service API       │
                    └──────────────┬──────────────────────┘
                                   │
                    ┌──────────────▼──────────────────────┐
                    │                                     │
                    │         DOMAIN / CORE               │
                    │   Entities, Value Objects, Logic    │
                    │                                     │
                    └──────────────┬──────────────────────┘
                                   │
                    ┌──────────────▼──────────────────────┐
                    │        OUTPUT PORTS (Interfaces)    │
                    │       Repository, Gateway APIs      │
                    └──────────────┬──────────────────────┘
                                   │ calls
                                   ▼
                    ┌─────────────────────────────────────┐
                    │         OUTPUT ADAPTERS             │
                    │  Database, External APIs, Cache     │
                    └─────────────────────────────────────┘
```

## Key Principles

### 1. Dependency Rule
- Dependencies point **inward** toward the domain
- Domain has **zero dependencies** on external layers
- Adapters depend on ports, ports depend on domain

### 2. Ports (Interfaces)
- **Input Ports**: Define what the application can do (use cases)
- **Output Ports**: Define what the application needs (repositories, gateways)
- Ports are owned by the domain layer

### 3. Adapters (Implementations)
- **Input Adapters** (Driving): Call the application (HTTP, CLI, events)
- **Output Adapters** (Driven): Called by the application (database, external APIs)
- Adapters implement or use port interfaces

## Domain Layer

The core of the application. Contains business logic with no external dependencies.

### Entities
- Business objects with identity and lifecycle
- Have unique identifier
- Contain business rules
- Can change state over time

### Value Objects
- Immutable objects defined by their attributes
- No identity, compared by value
- Immutable after creation
- Self-validating

### Domain Services
- Business logic that doesn't fit in a single entity
- Stateless operations
- Coordinate multiple entities
- Named after business operations

### Domain Events
- Record of something significant that happened
- Immutable
- Named in past tense
- Contain relevant data

## Application Layer

Orchestrates use cases by coordinating domain objects and ports.

### Use Cases / Application Services
One class per use case:
- CreateOrder
- ProcessPayment  
- CancelSubscription

Each use case:
- Receives input DTO
- Validates input
- Calls domain logic
- Uses output ports for persistence/external calls
- Returns output DTO

### Input/Output DTOs
Data Transfer Objects for crossing boundaries:
- Input DTOs: What the use case receives
- Output DTOs: What the use case returns
- Never expose domain entities directly

## File Organization

### One Use Case Per File
Each use case belongs in its own file. File name = use case name.

**BAD - Multiple use cases in one file:**
```
application/
└── auth.go          # Contains Login, Refresh, Logout - hard to find, grows over time
```

**GOOD - Separate files:**
```
application/
├── ports.go         # All port interfaces in one file
├── dto.go           # Shared DTOs (MessageResult, TokenResult)
├── login.go         # LoginUseCase + LoginCommand
├── refresh.go       # RefreshUseCase + RefreshCommand
├── logout.go        # LogoutUseCase + LogoutCommand
└── register.go      # RegisterUseCase + RegisterCommand
```

### Why Separate Files?
- **Single Responsibility**: Each file has one reason to change
- **Easy to locate**: File name matches use case name
- **Clean git history**: Changes to one use case don't touch others
- **Scales better**: Use cases can grow without affecting siblings

### Shared Types
Put shared types in dedicated files:
- `ports.go` - All port interfaces (Repository, Gateway, etc.)
- `dto.go` - Shared result types used by multiple use cases

### File Contents Pattern
Each use case file contains:
1. Command struct (input DTO)
2. UseCase struct
3. Constructor function
4. Execute method

```
// login.go
type LoginCommand struct { ... }      // Input
type LoginUseCase struct { ... }      // Use case
func NewLoginUseCase(...) { ... }     // Constructor
func (uc *LoginUseCase) Execute(...) { ... }  // Logic
```

## Ports

### Input Ports (Driving)
Interfaces that define application capabilities:
- UserService.CreateUser(input) -> output
- OrderService.PlaceOrder(input) -> output
- PaymentService.ProcessPayment(input) -> output

### Output Ports (Driven)
Interfaces that define external dependencies:
- UserRepository.Save(user)
- UserRepository.FindByID(id) -> user
- PaymentGateway.Charge(amount) -> result
- EmailSender.Send(email)
- EventPublisher.Publish(event)

## Adapters

### Input Adapters (Driving)
Entry points that call the application:
- HTTP/REST handlers
- gRPC handlers
- CLI commands
- Message consumers (SQS, Kafka)
- Scheduled jobs
- GraphQL resolvers

### Output Adapters (Driven)
Implementations of output ports:
- PostgresUserRepository implements UserRepository
- StripePaymentGateway implements PaymentGateway
- SESEmailSender implements EmailSender
- SNSEventPublisher implements EventPublisher

## Project Structure

```
project/
├── cmd/                      # Application entrypoints
│   └── lambda/
├── internal/
│   ├── domain/               # Core business logic
│   │   └── model/            # Entities, value objects, errors
│   ├── application/          # Use cases (one file per use case)
│   │   ├── ports.go          # Port interfaces
│   │   ├── dto.go            # Shared DTOs
│   │   ├── login.go          # LoginUseCase
│   │   └── register.go       # RegisterUseCase
│   └── adapters/
│       ├── input/            # Driving adapters (call the app)
│       │   ├── http/         # HTTP handlers
│       │   ├── lambda/       # Lambda handlers
│       │   └── grpc/         # gRPC handlers
│       └── output/           # Driven adapters (called by app)
│           ├── postgres/     # Database implementations
│           ├── cognito/      # Auth provider
│           └── ses/          # Email sender
└── go.mod
```

## Dependency Injection

Wire adapters to ports at application startup:

1. Create driven adapters (repositories, gateways)
2. Create application services with output ports
3. Create driving adapters with input ports
4. Start the application

## Testing Strategy

### Unit Tests (Domain)
- Test entities and value objects in isolation
- No mocks needed - pure business logic
- Fast, deterministic

### Unit Tests (Application)
- Test use cases with mocked output ports
- Verify orchestration logic
- Mock repositories and gateways

### Integration Tests (Adapters)
- Test driven adapters against real dependencies
- Use testcontainers or embedded databases
- Verify data mapping and queries

### End-to-End Tests
- Test driving adapters through to driven adapters
- Full flow verification
- Limited number, focused on critical paths

## Anti-Patterns to Avoid

### 1. Domain Depending on Frameworks
- BAD: Domain entity imports ORM annotations
- GOOD: Domain entity is plain object, adapter maps to ORM

### 2. Leaking Domain Entities
- BAD: HTTP handler returns domain entity directly
- GOOD: Use DTOs at boundaries, map in adapters

### 3. Business Logic in Adapters
- BAD: Validation or calculations in HTTP handler
- GOOD: All business logic in domain or application layer

### 4. Skipping Ports
- BAD: Use case directly instantiates repository implementation
- GOOD: Use case depends on repository interface (port)

### 5. Fat Use Cases
- BAD: Single use case with many responsibilities
- GOOD: One focused use case per business operation

## When to Use Hexagonal

**Good Fit:**
- Complex business logic
- Multiple entry points (API, CLI, events)
- Need to swap implementations (databases, providers)
- Long-lived projects requiring maintainability
- Teams practicing TDD

**May Be Overkill:**
- Simple CRUD applications
- Short-lived prototypes
- Very small services with trivial logic

## Quality Checklist

When implementing hexagonal architecture:
- [ ] Domain has zero external dependencies
- [ ] All external access goes through ports
- [ ] Each adapter implements exactly one port
- [ ] **One use case per file** (file name = use case name)
- [ ] Shared DTOs in separate `dto.go` file
- [ ] Use cases are single-purpose
- [ ] DTOs used at all boundaries
- [ ] Domain entities never leak to adapters
- [ ] Dependency injection wires components
- [ ] Tests follow the testing pyramid
