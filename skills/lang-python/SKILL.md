---
name: lang-python
description: Apply Python best practices, idiomatic patterns, and modern Python features when writing or reviewing Python code.
---

# Python Skill

Apply these Python patterns and practices when working with Python code.

## Code Style

- Follow PEP 8 style guide
- Use type hints (Python 3.9+)
- Prefer f-strings for formatting
- Use `pathlib` over `os.path`
- Maximum line length: 88 (Black default)

## Type Hints

### Basic Types
```python
from typing import Optional

def greet(name: str) -> str:
    return f"Hello, {name}"

def find_user(user_id: int) -> Optional[User]:
    return users.get(user_id)

def process_items(items: list[str]) -> dict[str, int]:
    return {item: len(item) for item in items}
```

### Complex Types
```python
from typing import TypeVar, Callable, TypedDict
from collections.abc import Iterable, Sequence

T = TypeVar("T")

def first(items: Sequence[T]) -> T | None:
    return items[0] if items else None

class UserDict(TypedDict):
    id: str
    email: str
    name: str | None

Handler = Callable[[dict], dict]
```

### Protocols
```python
from typing import Protocol

class Repository(Protocol):
    def get(self, id: str) -> dict | None: ...
    def save(self, item: dict) -> None: ...

def process(repo: Repository) -> None:
    item = repo.get("123")
    if item:
        repo.save(item)
```

## Data Classes

### Basic Dataclass
```python
from dataclasses import dataclass, field
from datetime import datetime

@dataclass
class User:
    id: str
    email: str
    created_at: datetime = field(default_factory=datetime.utcnow)
    roles: list[str] = field(default_factory=list)

@dataclass(frozen=True)
class Config:
    api_url: str
    timeout: int = 30
```

### Pydantic Models
```python
from pydantic import BaseModel, Field, EmailStr

class CreateUserRequest(BaseModel):
    email: EmailStr
    name: str = Field(min_length=1, max_length=100)

    class Config:
        extra = "forbid"

class User(BaseModel):
    id: str
    email: EmailStr
    name: str

    class Config:
        from_attributes = True
```

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

## Lambda Handler Pattern

```python
import json
import logging
from typing import Any

logger = logging.getLogger()
logger.setLevel(logging.INFO)

def handler(event: dict, context: Any) -> dict:
    try:
        body = json.loads(event.get("body", "{}"))
        result = process_request(body)
        return {
            "statusCode": 200,
            "body": json.dumps(result),
        }
    except ValidationError as e:
        logger.warning(f"Validation error: {e}")
        return {
            "statusCode": 400,
            "body": json.dumps({"error": str(e)}),
        }
    except Exception as e:
        logger.exception("Unexpected error")
        return {
            "statusCode": 500,
            "body": json.dumps({"error": "Internal server error"}),
        }
```

## Testing

### Pytest Basics
```python
import pytest
from unittest.mock import Mock, patch

def test_create_user():
    user = create_user("test@example.com", name="Test")
    assert user.email == "test@example.com"
    assert user.name == "Test"

def test_user_not_found():
    with pytest.raises(NotFoundError) as exc_info:
        get_user("invalid-id")
    assert exc_info.value.code == "NOT_FOUND"

@pytest.fixture
def mock_db():
    with patch("myapp.database.get_connection") as mock:
        yield mock

def test_with_mock_db(mock_db):
    mock_db.return_value.execute.return_value = [{"id": "1"}]
    result = fetch_data()
    assert len(result) == 1
```

### Parametrized Tests
```python
@pytest.mark.parametrize("input,expected", [
    ("hello", "HELLO"),
    ("world", "WORLD"),
    ("", ""),
])
def test_uppercase(input: str, expected: str):
    assert input.upper() == expected
```

## Project Structure

```
project/
├── src/
│   └── myapp/
│       ├── __init__.py
│       ├── main.py
│       ├── domain/
│       │   ├── __init__.py
│       │   └── models.py
│       ├── handlers/
│       │   ├── __init__.py
│       │   └── api.py
│       └── repository/
│           ├── __init__.py
│           └── dynamodb.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   └── test_handlers.py
├── pyproject.toml
└── requirements.txt
```

## Quality Checklist

When writing Python code, verify:
- [ ] Type hints on all public functions
- [ ] No bare `except:` clauses
- [ ] Resources properly closed (context managers)
- [ ] No mutable default arguments
- [ ] Tests cover happy path and error cases
- [ ] Logging instead of print statements
- [ ] No hardcoded configuration
- [ ] Dependencies pinned in requirements.txt
