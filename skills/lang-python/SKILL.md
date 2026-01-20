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
