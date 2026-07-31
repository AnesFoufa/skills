# Python Design — Parse at the Edge

## Parse, don't validate

Bad (validate): check a predicate, return bool, caller must remember to check.
Good (parse): convert raw input into a type that *cannot* be wrong.

```python
# BAD
def is_valid_call_id(s: str) -> bool:
    return len(s) > 0

# GOOD
CallId = NewType('CallId', str)

class RawCallRequest(BaseModel):   # Pydantic at the edge
    call_id: str
    gateway: str

    def to_domain(self) -> CallRequest:
        return CallRequest(
            call_id=CallId(self.call_id),
            gateway=GatewayName(self.gateway),
        )
```

## Parse at the edge — Pydantic

Use Pydantic models at every system boundary:
- **HTTP**: FastAPI request body / query params -> Pydantic model -> `.to_domain()`
- **DB row**: raw `dict` from cursor -> Pydantic model (or dataclass with `from_row`)
- **Queue message**: raw JSON -> Pydantic model -> domain type

The core (`core/`, `service/`) never receives `dict`, `str`, or raw `int`.

```python
# Edge (router layer)
@router.post("/calls")
def create_call(body: RawCallRequest) -> Response:
    request = body.to_domain()         # parsed, typed, valid
    service.handle(request)            # core sees CallRequest, not dict
```

## Smart constructors

Use when Pydantic cannot express the invariant (e.g. cross-field constraint, external lookup).

```python
@dataclass(frozen=True)
class DateRange:
    start: date
    end: date

    @classmethod
    def create(cls, start: date, end: date) -> "DateRange | None":
        if end < start:
            return None             # caller handles the failure case
        return cls(start=start, end=end)
```

Rules:
- Private `__init__` pattern if you want to prevent direct construction.
- Return `Self | None` or a `Result` type; never raise in the factory.
- `__init__` may raise only if the type is purely internal (not exposed at a boundary).

## NewType for domain primitives

```python
from typing import NewType

CallId       = NewType('CallId', str)
GatewayName  = NewType('GatewayName', str)
DurationMs   = NewType('DurationMs', int)

# Function signatures become self-documenting and catch argument-order bugs:
def find_call(call_id: CallId, gateway: GatewayName) -> Call | None: ...

# mypy rejects this:
find_call(gateway, call_id)   # error: expected CallId, got GatewayName
```

Use `NewType` (not `TypeAlias`) when the primitive should be distinguishable to the type checker.
Use `TypeAlias` only for complex generic aliases (e.g. `Rows = list[dict[str, Any]]`).

---

## Error handling

### Result type (roll your own, no deps)

```python
from dataclasses import dataclass
from typing import Generic, TypeVar, assert_never

T = TypeVar('T')
E = TypeVar('E')

@dataclass(frozen=True)
class Ok(Generic[T]):
    value: T

@dataclass(frozen=True)
class Err(Generic[E]):
    error: E

Result = Ok[T] | Err[E]
```

Usage in the domain core:

```python
@dataclass(frozen=True)
class InvalidDateRange:
    start: date
    end: date

@dataclass(frozen=True)
class DateRange:
    start: date
    end: date

    @classmethod
    def create(cls, start: date, end: date) -> "Ok[DateRange] | Err[InvalidDateRange]":
        if end < start:
            return Err(InvalidDateRange(start=start, end=end))
        return Ok(cls(start=start, end=end))

# Caller must handle both branches:
match DateRange.create(start, end):
    case Ok(value=r): schedule(r)
    case Err(error=e): return bad_request(e)
    case _ as u: assert_never(u)
```

### Exception boundary — catch once at the shell

```python
# worker/consolidation_worker.py  (imperative shell)
def run_once(self) -> None:
    try:
        cdrs = self._repo.fetch_pending()       # may raise infrastructure exceptions
        rows = build_history_rows(cdrs, datetime.now(UTC))  # pure, never raises
        self._repo.insert(rows)                 # may raise infrastructure exceptions
    except OperationalError as e:
        logger.error("DB failure: %s", e)       # log, retry or fail
        raise
    # No bare `except Exception` — let programming errors crash loudly
```

Rule: if you find yourself catching a domain error as an exception deep in the call stack, it should have been a return type instead.
