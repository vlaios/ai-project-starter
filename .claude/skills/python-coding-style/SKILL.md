---
name: python-coding-style
description: Apply personal Python coding standards and preferences when writing, reviewing, or refactoring Python code. Use this skill whenever the user asks to write Python code, create a Python module, refactor existing Python, scaffold a project, or review Python for style. Also trigger when the user says "write me a Python...", "create a Python class/function/module/service", "scaffold a Python project", or any Python code generation task, even if they don't explicitly mention style. This skill ensures all output matches the user's exact personal preferences for formatting, architecture, tooling, and documentation.
---

# Python Coding Style: Personal Preferences

This skill encodes the exact Python coding standards and architectural preferences to apply
whenever writing, generating, or reviewing Python code for this user.

**Always apply this skill when generating any Python code, no exceptions.**

---

## Code Style

### Type Hints
- Use type hints **everywhere**: function parameters, return types, local variables, class attributes
- Use `from __future__ import annotations` for forward references
- Use `X | None` over `Optional[X]` (Python 3.10+)
- Use `list[str]` over `List[str]`, `dict[str, int]` over `Dict[str, int]` (Python 3.9+)

```python
# CORRECT
def process_items(items: list[str], limit: int = 10) -> dict[str, int]:
    result: dict[str, int] = {}
    ...

# WRONG: missing hints, old-style generics
def process_items(items, limit=10):
    result = {}
    ...
```

### Formatting
- **Line length**: 88 characters (Black default)
- **Quotes**: Defer to Black (double quotes by default)
- **Trailing commas**: Always in multi-line structures, improves diffs
- **Formatter**: Black + ruff, never fight the formatter, just run it

```python
# CORRECT: trailing commas, Black-formatted
user = User(
    name="Alice",
    email="alice@example.com",
    role="admin",
)

# WRONG: no trailing comma
user = User(
    name="Alice",
    email="alice@example.com",
    role="admin"
)
```

### Docstrings: Google Style
- Every public function, class, and method gets a Google-style docstring
- Private/internal helpers (`_name`) only need docstrings if non-obvious
- Never write docstrings that just restate the function name

```python
def fetch_user(user_id: str, include_deleted: bool = False) -> User | None:
    """Fetch a user record by ID from the repository.

    Args:
        user_id: The unique identifier of the user.
        include_deleted: Whether to include soft-deleted users.

    Returns:
        The User object if found, None otherwise.

    Raises:
        RepositoryError: If the database connection fails.
    """
```

---

## Architecture

### Design Philosophy: Hybrid
- **Domain models → Pydantic classes**: validation, serialization, type safety baked in
- **Business logic → pure functions**: stateless, testable, composable
- **Infrastructure → classes with DI**: repositories, services, clients

```python
# Domain model: Pydantic
class Order(BaseModel):
    id: str
    user_id: str
    items: list[OrderItem]
    status: OrderStatus = OrderStatus.PENDING

# Business logic: pure function
def calculate_total(order: Order, discount: Discount | None = None) -> Decimal:
    subtotal: Decimal = sum(item.price * item.quantity for item in order.items)
    if discount:
        return apply_discount(subtotal, discount)
    return subtotal

# Infrastructure: class with DI
class OrderRepository:
    def __init__(self, db: DatabaseClient) -> None:
        self._db = db

    async def get_by_id(self, order_id: str) -> Order | None:
        ...
```

### Error Handling: Mixed Strategy
- **Recoverable errors**: handle locally with try/except, return sensible defaults or `None`
- **Fatal/unexpected errors**: define a custom exception hierarchy, let them bubble to a central handler
- **Never swallow exceptions silently**: always log or re-raise

```python
# Custom exception hierarchy: define at domain level
class AppError(Exception):
    """Base application error."""

class RepositoryError(AppError):
    """Database/storage layer failure."""

class ValidationError(AppError):
    """Domain validation failure."""

# Local: recoverable
def parse_date(raw: str) -> datetime | None:
    try:
        return datetime.fromisoformat(raw)
    except ValueError:
        logger.warning("Invalid date format: {}", raw)
        return None

# Central: fatal errors bubble up to here
def handle_error(error: Exception) -> None:
    if isinstance(error, AppError):
        logger.error("Application error: {}", error)
    else:
        logger.critical("Unexpected error: {}", error)
        raise
```

### Async Strategy
- Use `async/await` **only** for IO-bound operations: HTTP calls, DB queries, file IO
- Keep CPU-bound logic synchronous
- Never mix sync blocking calls inside async functions, use `asyncio.to_thread()` if needed

```python
# CORRECT: async for IO
async def fetch_user_data(user_id: str) -> UserData:
    async with httpx.AsyncClient() as client:
        response = await client.get(f"/users/{user_id}")
    return UserData.model_validate(response.json())

# CORRECT: sync for pure logic
def compute_risk_score(features: list[float]) -> float:
    return sum(f * w for f, w in zip(features, WEIGHTS))
```

### Patterns to Always Apply

**Dependency Injection**
```python
# CORRECT: inject dependencies, never instantiate inside
class UserService:
    def __init__(self, repo: UserRepository, mailer: EmailClient) -> None:
        self._repo = repo
        self._mailer = mailer
```

**Repository Pattern**
```python
# Keep all data access behind repository interfaces
class UserRepository(Protocol):
    async def get(self, user_id: str) -> User | None: ...
    async def save(self, user: User) -> None: ...
    async def delete(self, user_id: str) -> None: ...
```

**Factory Functions over Constructors**
```python
# CORRECT: factory function for complex init
def create_http_client(settings: Settings) -> httpx.AsyncClient:
    return httpx.AsyncClient(
        base_url=settings.api_base_url,
        timeout=settings.timeout_seconds,
        headers={"Authorization": f"Bearer {settings.api_key}"},
    )
```

**Context Managers for Resources**
```python
# CORRECT: always use context managers for resources
async with asyncpg.create_pool(dsn=settings.db_url) as pool:
    async with pool.acquire() as conn:
        result = await conn.fetch(query)
```

---

## Data Modeling: Pydantic

- Use `BaseModel` for all domain data structures
- Use `BaseSettings` for all config and environment variables
- Use `model_validator` and `field_validator` for business rule validation
- Prefer `model_config = ConfigDict(frozen=True)` for value objects

```python
from pydantic import BaseModel, ConfigDict, field_validator
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    model_config = ConfigDict(env_file=".env", env_file_encoding="utf-8")

    api_key: str
    db_url: str
    debug: bool = False
    max_retries: int = 3

class UserProfile(BaseModel):
    model_config = ConfigDict(frozen=True)

    user_id: str
    email: str
    age: int

    @field_validator("age")
    @classmethod
    def validate_age(cls, v: int) -> int:
        if v < 0 or v > 150:
            raise ValueError(f"Invalid age: {v}")
        return v
```

---

## Logging: Loguru

- Use `loguru` exclusively, no `print()`, no `logging` module
- Log at the right level: `debug` for internals, `info` for milestones, `warning` for recoverable issues, `error` for failures, `critical` for system-level failures
- Use structured logging with named placeholders `{}`, not f-strings in log calls
- Configure once at app entry point in `core/logging.py`

```python
from loguru import logger

# core/logging.py: configure once
def setup_logging(level: str = "INFO") -> None:
    """Configure loguru for the application.

    Args:
        level: Minimum log level to capture.
    """
    logger.add(
        "logs/app.log",
        rotation="10 MB",
        retention="7 days",
        level=level,
        format="{time:YYYY-MM-DD HH:mm:ss} | {level} | {name}:{function}:{line} | {message}",
    )

# Usage: named placeholders, not f-strings
logger.info("Processing order {order_id} for user {user_id}", order_id=order.id, user_id=order.user_id)
logger.error("Failed to fetch user {user_id}: {error}", user_id=user_id, error=str(e))
```

---

## File & Project Organization

### Structure: Domain/Feature Grouped
```
src/
├── orders/
│   ├── __init__.py
│   ├── models.py        # Pydantic models
│   ├── repository.py    # Data access
│   ├── service.py       # Business logic functions
│   └── router.py        # API layer (if applicable)
├── users/
│   ├── __init__.py
│   ├── models.py
│   ├── repository.py
│   └── service.py
├── core/
│   ├── config.py        # Pydantic BaseSettings
│   ├── exceptions.py    # Custom exception hierarchy
│   └── logging.py       # Loguru setup
└── main.py
```

### Config: Pydantic BaseSettings Only
- No hardcoded constants in business logic
- No raw `os.environ.get()` calls scattered around
- All config flows through a single `Settings` instance
- Use `.env` files for local dev, real env vars in production

### Imports: Let ruff/isort Handle It
- Run `ruff check --fix` and `ruff format`, never manually order imports
- Use absolute imports always, no relative imports except within the same package's `__init__.py`

---

## Comments

- **Minimal**: clean, well-named code is self-documenting
- Add inline comments **only** for:
  - Non-obvious algorithmic logic
  - Workarounds for external library bugs/quirks
  - Business rule rationale that can't be expressed in code
- **Never** comment what the code does, comment **why** when it's not obvious

```python
# CORRECT: explains the why
# Retry with exponential backoff: the upstream API rate-limits at 100 req/min
for attempt in range(max_retries):
    ...

# WRONG: restates the code
# Loop through items
for item in items:
    ...
```

---

## Testing: pytest

- Every module gets a corresponding `tests/test_<module>.py`
- Use `pytest` fixtures for dependency injection in tests
- Test **behavior**, not implementation
- Use `pytest.mark.parametrize` for data-driven cases
- Mock at the boundary (repositories, HTTP clients), never mock domain logic

```python
import pytest
from unittest.mock import AsyncMock

@pytest.fixture
def mock_repo() -> UserRepository:
    """Provide a mock UserRepository for testing."""
    repo = AsyncMock(spec=UserRepository)
    repo.get.return_value = User(id="123", name="Alice")
    return repo

@pytest.mark.parametrize("age,expected", [
    (25, True),
    (-1, False),
    (200, False),
])
def test_validate_age(age: int, expected: bool) -> None:
    ...
```

---

## Toolchain

Standard setup for every project:

| Tool | Purpose | Config file |
|---|---|---|
| `ruff` | Linting + formatting | `pyproject.toml` |
| `mypy` | Static type checking | `pyproject.toml` |
| `bandit` | Security scanning | `pyproject.toml` |
| `pre-commit` | Enforce all of the above on commit | `.pre-commit-config.yaml` |
| `pytest` | Testing | `pyproject.toml` |
| `loguru` | Logging | `core/logging.py` |
| `pydantic` | Models + settings | Throughout |

### pyproject.toml

> Version values below are placeholders. Verify the latest stable Python on python.org before pinning.
```toml
[tool.ruff]
line-length = 88
target-version = "<py3XX: latest stable>"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "UP", "S", "B", "A"]

[tool.mypy]
strict = true
python_version = "<3.XX: latest stable>"

[tool.bandit]
skips = ["B101"]  # Allow assert in tests

[tool.pytest.ini_options]
testpaths = ["tests"]
asyncio_mode = "auto"
```

### .pre-commit-config.yaml

> Hook revisions below are placeholders. Run `pre-commit autoupdate` to pin each hook to its latest release.
```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: <latest-stable>
    hooks:
      - id: ruff
        args: [--fix]
      - id: ruff-format
  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: <latest-stable>
    hooks:
      - id: mypy
  - repo: https://github.com/PyCQA/bandit
    rev: <latest-stable>
    hooks:
      - id: bandit
        args: ["-c", "pyproject.toml"]
```
