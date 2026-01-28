# Python Type Hints & Data Classes

## Basic Types

```python
from typing import Optional

def greet(name: str) -> str:
    return f"Hello, {name}"

def find_user(user_id: int) -> Optional[User]:
    return users.get(user_id)

def process_items(items: list[str]) -> dict[str, int]:
    return {item: len(item) for item in items}
```

## Complex Types

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

## Protocols

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

## Basic Dataclass

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

## Pydantic Models

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
