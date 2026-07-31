# Modeling Patterns

Use these examples when the representation choice is the main design question. Examples target Python 3.11 or newer.

## Static identity versus runtime validation

Use `NewType` to stop accidental mixing during static checking. It does not validate or wrap values at runtime.

```python
from typing import NewType

UserId = NewType("UserId", str)
OrderId = NewType("OrderId", str)

def load_order(order_id: OrderId) -> None:
    pass

user_id = UserId("user-1")
# A static checker rejects: load_order(user_id)
```

Use a value object when construction must enforce an invariant.

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class NonEmptyName:
    value: str

    def __post_init__(self) -> None:
        if not self.value.strip():
            raise ValueError("name must not be empty")
```

Choose a factory returning a result instead when invalid input is an ordinary branch that callers should handle explicitly.

## Model lifecycle states without partial objects

Split types when fields become mandatory after a transition.

```python
from dataclasses import dataclass
from datetime import datetime

@dataclass(frozen=True)
class DraftReport:
    body: str

@dataclass(frozen=True)
class PublishedReport:
    body: str
    published_at: datetime

def publish(report: DraftReport, now: datetime) -> PublishedReport:
    return PublishedReport(body=report.body, published_at=now)
```

Keep one type with an optional field when absence really is a stable, meaningful state and splitting the type would make consumers harder to understand.

## Select among enum, union, and class hierarchy

- Use an enum for closed named constants whose variants carry no different fields.
- Use a union of dataclasses when variants carry different data and consumers benefit from exhaustive matching.
- Use a class hierarchy when behavior inheritance, plugin extensibility, or runtime polymorphism is the actual requirement.
- Do not call an ordinary ABC hierarchy sealed. Use an explicit union for the closed set consumed by a function.

## Apply immutability proportionally

Prefer frozen dataclasses for values, commands, events, and function results. Allow controlled mutation for identity-bearing entities, ORM-managed objects, performance-sensitive accumulators, and builders. Keep mutation local and preserve invariants at each public operation.
