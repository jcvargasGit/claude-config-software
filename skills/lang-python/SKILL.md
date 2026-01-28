---
name: lang-python
description: Python language expert for *.py files. Covers Python versions, packages (pip, conda), type hints, async/await, testing (pytest), and Pythonic patterns. Use for any Python question.
model: opus
---

# Python Skill

Apply these Python patterns and practices when working with Python code.

## Additional Resources

- [Type Hints & Data Classes](./types.md) - Type hints, protocols, dataclasses, Pydantic
- [Patterns](./patterns.md) - Functions, async, error handling, collections
- [Testing](./testing.md) - pytest basics, fixtures, parametrized tests

## Code Style

- Follow PEP 8 style guide
- Use type hints (Python 3.9+)
- Prefer f-strings for formatting
- Use `pathlib` over `os.path`
- Maximum line length: 88 (Black default)
- **All imports must be at the top of the file** - never inside functions or methods

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

### Test Style by Level

| Level | Style | Python Pattern |
|-------|-------|----------------|
| **Unit** | Simple, pytest | Direct assertions |
| **Integration/E2E** | BDD Given/When/Then | Step context classes |

---

### Unit Tests: Simple (pytest)

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
```

### Fixtures for Setup/Teardown

```python
# conftest.py - shared fixtures
import pytest

@pytest.fixture
def db_connection():
    conn = create_connection()
    yield conn
    conn.close()

@pytest.fixture
def test_user(db_connection):
    user = create_test_user(db_connection)
    yield user
    delete_test_user(db_connection, user.id)
```

### Markers for Test Categories

```python
# pytest.ini
# [pytest]
# markers =
#     integration: marks tests as integration tests
#     slow: marks tests as slow

@pytest.mark.integration
def test_cognito_integration():
    # Only runs with: pytest -m integration
    pass

@pytest.mark.slow
def test_heavy_computation():
    pass
```

---

### Integration/E2E Tests: BDD Step Pattern

```python
# steps/given.py
class GivenContext:
    def __init__(self, db):
        self.db = db

    def a_user_exists(self) -> User:
        return create_test_user(self.db)

# steps/when.py
class WhenContext:
    def __init__(self, client):
        self.client = client

    def call_login(self, email: str, password: str) -> Response:
        return self.client.post("/login", json={"email": email, "password": password})

# steps/then.py
class ThenContext:
    def status_code_is(self, response: Response, expected: int):
        assert response.status_code == expected

    def response_contains_token(self, response: Response):
        data = response.json()
        assert "access_token" in data

# test_login.py
def test_login_success(given, when, then):
    # Given
    user = given.a_user_exists()

    # When
    response = when.call_login(user.email, "password")

    # Then
    then.status_code_is(response, 200)
    then.response_contains_token(response)
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

### Mock Patterns

```python
from unittest.mock import Mock, patch, AsyncMock

@pytest.fixture
def mock_db():
    with patch("myapp.database.get_connection") as mock:
        yield mock

def test_with_mock_db(mock_db):
    mock_db.return_value.execute.return_value = [{"id": "1"}]
    result = fetch_data()
    assert len(result) == 1

# Async mock
@pytest.fixture
def mock_api():
    with patch("myapp.api.fetch_user", new_callable=AsyncMock) as mock:
        mock.return_value = {"id": "1", "email": "test@example.com"}
        yield mock
```

## Observability

### Structured Logging with structlog

```python
import structlog

structlog.configure(
    processors=[
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.JSONRenderer()
    ]
)

logger = structlog.get_logger(service="my-service", stage=os.environ.get("STAGE"))

# Info level
logger.info("request handled", method="POST", path="/users", status=200, duration=0.042)

# Warn for 4xx
logger.warning("request failed", error=str(err), operation="login", status=401)

# Error for 5xx
logger.error("request failed", error=str(err), operation="create_user", status=500)
```

### Error Logging Helper

```python
def log_error(err: Exception, operation: str) -> dict:
    status = map_error_to_status(err)
    level = "error" if status >= 500 else "warning"

    getattr(logger, level)(
        "request failed",
        error=str(err),
        operation=operation,
        status=status
    )

    return error_response(err)

# Usage
try:
    result = process_request(data)
except AppError as e:
    return log_error(e, "register")
```

### Context-Aware Logging

```python
from contextvars import ContextVar

request_id: ContextVar[str] = ContextVar("request_id", default="")

def get_logger():
    return logger.bind(request_id=request_id.get())

# Middleware
def request_middleware(request, call_next):
    request_id.set(request.headers.get("x-request-id", str(uuid4())))
    return call_next(request)
```

## Quality Checklist

When writing Python code, verify:
- [ ] All imports at the top of the file (not inside functions/methods)
- [ ] Type hints on all public functions
- [ ] No bare `except:` clauses
- [ ] Resources properly closed (context managers)
- [ ] No mutable default arguments
- [ ] Tests cover happy path and error cases
- [ ] Logging instead of print statements
- [ ] No hardcoded configuration
- [ ] Dependencies pinned in requirements.txt
