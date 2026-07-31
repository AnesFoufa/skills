---
name: python-design
description: "Apply domain modeling principles when designing types, DB schemas, and service layers in Python: strong typing, ADTs with pattern matching over inheritance, ABC/Protocol interfaces, minimal nullability, make illegal states non-representable at type and DB level, functional core / imperative shell. Use when designing or reviewing non-trivial Python code — models, schemas, service layers, interfaces, or domain logic. Not for simple scripts or one-off tooling."
---

# Python Design Principles

## 1. Algebraic data types over inheritance

Model variants as a sealed ABC + **frozen** dataclasses. Use `match` to dispatch.

```python
from abc import ABC
from dataclasses import dataclass
from typing import assert_never

class CallEvent(ABC): ...

@dataclass(frozen=True)          # frozen = immutable, always
class CallStarted(CallEvent):
    call_id: CallId
    started_at: datetime

@dataclass(frozen=True)
class CallEnded(CallEvent):
    call_id: CallId
    ended_at: datetime
    duration_s: DurationS        # non-nullable: computed at END, not optional

def handle(event: CallEvent) -> None:
    match event:
        case CallStarted(call_id=cid, started_at=t): ...
        case CallEnded(call_id=cid, ended_at=t, duration_s=d): ...
        case _ as unreachable: assert_never(unreachable)  # exhaustiveness
```

Rules:
- Prefer `frozen=True` on domain dataclasses — mutable domain objects contradict functional core. Justify any exception explicitly (e.g. incremental construction via a builder).
- `assert_never` in every `match` on an ADT — mypy errors when a new variant is unhandled.

## 2. Make illegal states non-representable

At the **type level**: no `Optional` unless absence is a meaningful domain concept.
At the **DB level**: `NOT NULL`, `CHECK`, `ENUM`, FK constraints — not runtime guards.

```python
# BAD: nullable escape hatch
@dataclass
class HistoryRow:
    consolidated_at: datetime | None  # can be None? why? when? what enforces it?

# GOOD: always present, injected at creation
@dataclass(frozen=True)
class HistoryRow:
    consolidated_at: datetime          # set at projection time, never absent
```

```sql
-- BAD
consolidated_at TIMESTAMP NULL

-- GOOD
consolidated_at TIMESTAMP NOT NULL
```

## 3. ABC and Protocol for architectural boundaries

Use **`ABC`** when implementations must explicitly opt in (repository ports, adapters). Use **`Protocol`** when you want structural subtyping — the implementor doesn't need to import or inherit from the interface, which keeps modules decoupled.

```python
from abc import ABC, abstractmethod
from typing import Protocol

# ABC: explicit contract — implementors inherit and register
class Repository(ABC):
    @abstractmethod
    def save(self, entity: Entity) -> None: ...

# Protocol: structural — any class with a matching signature satisfies it
class Saveable(Protocol):
    def save(self, entity: Entity) -> None: ...
```

Choose ABC when you control both sides of the boundary and want mypy to enforce the contract at definition time. Choose Protocol when the implementor is in a different package or you want to avoid circular imports.

Rule: if you can't name the architectural layer the interface lives in, you don't need one — use a concrete type.

## 4. Functional core, imperative shell

```
core/           <- pure functions, frozen dataclasses, no I/O
  logic.py      <- process(inputs, now: datetime) -> list[Result]
service/        <- thin orchestrator, injects side-effectful values
  service.py    <- results = process(inputs, datetime.now(UTC))
adapters/       <- imperative shell: DB, time, network, queues
  repository.py <- repo.save(results)
```

Rule: if a function calls `datetime.now()`, reads from DB, or writes to disk, it belongs in the shell. The core must accept these values as parameters.

## 5. Minimal nullable fields checklist

Applies equally to: dataclasses, Pydantic models, DB columns.

Before adding `Optional` or a nullable column, ask:

```
[ ] Is absence a real domain state (not "not loaded yet" or "unknown")?
[ ] If absent only during creation, can I enforce presence at the constructor?
[ ] Can I split the type into two (before/after) instead of making a field optional?
[ ] Can the DB constraint enforce this instead of application code?
```

If all answers are "no", keep it non-nullable.

## 6. Parse at the edge

**Pydantic** at system boundaries (HTTP, DB row, queue message) — parse raw data into domain types before it enters the core. The core receives already-valid types, never raw `dict` / `str` / `int`.

**`NewType`** over bare primitives — gives distinct type identity, prevents wrong-argument-order bugs:

```python
from typing import NewType
UserId = NewType('UserId', str)
OrderId = NewType('OrderId', str)
# get_order(UserId(...))  <- mypy error: expected OrderId
```

**Smart constructors** for invariants Pydantic can't express (e.g. non-empty strings, validated ranges). `NewType` and smart constructors are complementary, not alternatives:

```python
@dataclass(frozen=True)
class Email:
    value: str
    def __post_init__(self):
        if '@' not in self.value:
            raise ValueError(f"Invalid email: {self.value}")
```

## 7. Error handling

- **Domain functions never raise.** Expected failures are return types.
- `T | None` — for simple not-found where absence is self-explanatory.
- `Result[T, E]` — when the caller needs to know *why* it failed, or there are multiple failure modes.
- **Exceptions only for:** programming errors (bugs) and infrastructure failures (DB down, network).
- Never model domain variants as exception subclasses — that is inheritance-based polymorphism for errors, the same antipattern as class inheritance in data modeling.
- Catch infrastructure exceptions **once**, at the outermost shell layer. Convert to an error return type or log and fail.

```
Pydantic ValidationError  <- caught by router       -> 422
Infrastructure exception  <- caught by shell/worker  -> 500 / retry
Domain errors             <- return type (never raise)
```
