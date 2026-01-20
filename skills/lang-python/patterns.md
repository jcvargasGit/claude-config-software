# Python Patterns

## Functions

### Default Arguments
```python
def fetch_users(
    limit: int = 10,
    offset: int = 0,
    active_only: bool = True,
) -> list[User]:
    query = build_query(active_only)
    return query.limit(limit).offset(offset).all()
```

### Keyword-Only Arguments
```python
def create_user(
    email: str,
    *,
    name: str | None = None,
    send_welcome: bool = True,
) -> User:
    user = User(email=email, name=name)
    if send_welcome:
        send_welcome_email(user)
    return user
```

### Context Managers
```python
from contextlib import contextmanager

@contextmanager
def database_transaction():
    conn = get_connection()
    try:
        yield conn
        conn.commit()
    except Exception:
        conn.rollback()
        raise
    finally:
        conn.close()

# Usage
with database_transaction() as conn:
    conn.execute(query)
```

## Async Patterns

### Async/Await
```python
import asyncio
import httpx

async def fetch_user(client: httpx.AsyncClient, user_id: str) -> dict:
    response = await client.get(f"/users/{user_id}")
    response.raise_for_status()
    return response.json()

async def fetch_all_users(user_ids: list[str]) -> list[dict]:
    async with httpx.AsyncClient() as client:
        tasks = [fetch_user(client, uid) for uid in user_ids]
        return await asyncio.gather(*tasks)
```

### Async Context Manager
```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def get_db_session():
    session = await create_session()
    try:
        yield session
        await session.commit()
    except Exception:
        await session.rollback()
        raise
    finally:
        await session.close()
```

## Error Handling

### Custom Exceptions
```python
class AppError(Exception):
    def __init__(self, message: str, code: str):
        super().__init__(message)
        self.code = code

class NotFoundError(AppError):
    def __init__(self, resource: str, id: str):
        super().__init__(f"{resource} not found: {id}", "NOT_FOUND")
        self.resource = resource
        self.id = id

class ValidationError(AppError):
    def __init__(self, field: str, message: str):
        super().__init__(f"{field}: {message}", "VALIDATION_ERROR")
        self.field = field
```

### Result Pattern
```python
from dataclasses import dataclass
from typing import Generic, TypeVar

T = TypeVar("T")
E = TypeVar("E")

@dataclass
class Ok(Generic[T]):
    value: T

@dataclass
class Err(Generic[E]):
    error: E

Result = Ok[T] | Err[E]

def parse_int(value: str) -> Result[int, str]:
    try:
        return Ok(int(value))
    except ValueError:
        return Err(f"Invalid integer: {value}")
```

## Collections

### Comprehensions
```python
# List comprehension
active_users = [u for u in users if u.active]

# Dict comprehension
user_by_id = {u.id: u for u in users}

# Set comprehension
unique_emails = {u.email.lower() for u in users}

# Generator expression
total = sum(order.total for order in orders)
```

### Iteration Patterns
```python
from itertools import groupby, chain
from operator import attrgetter

# Enumerate
for i, item in enumerate(items):
    print(f"{i}: {item}")

# Zip
for user, score in zip(users, scores, strict=True):
    print(f"{user.name}: {score}")

# Group by
sorted_users = sorted(users, key=attrgetter("department"))
for dept, group in groupby(sorted_users, key=attrgetter("department")):
    print(f"{dept}: {list(group)}")
```
