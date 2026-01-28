# Python Testing

## Pytest Basics

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

## Parametrized Tests

```python
@pytest.mark.parametrize("input,expected", [
    ("hello", "HELLO"),
    ("world", "WORLD"),
    ("", ""),
])
def test_uppercase(input: str, expected: str):
    assert input.upper() == expected
```

## Fixtures

```python
import pytest

@pytest.fixture
def sample_user():
    return User(id="123", email="test@example.com", name="Test")

@pytest.fixture
def mock_repository():
    repo = Mock()
    repo.find_by_id.return_value = None
    return repo

@pytest.fixture(autouse=True)
def reset_database():
    """Runs before each test automatically"""
    setup_test_db()
    yield
    teardown_test_db()
```

## Async Testing

```python
import pytest

@pytest.mark.asyncio
async def test_async_fetch():
    result = await fetch_user("123")
    assert result.id == "123"
```

## conftest.py

```python
# tests/conftest.py
import pytest

@pytest.fixture(scope="session")
def database_url():
    return "postgresql://test:test@localhost/test"

@pytest.fixture
def client(database_url):
    app = create_app(database_url)
    with app.test_client() as client:
        yield client
```
