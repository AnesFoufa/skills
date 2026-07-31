---
name: python-design
description: "Design or review non-trivial Python domain models, schemas, service layers, and architectural boundaries. Use when choosing domain types, modeling lifecycle states or variants, defining ABC or Protocol ports, handling expected errors, reducing nullability, parsing untrusted input, separating deterministic logic from I/O, or aligning application and database invariants. Apply these patterns contextually; do not use for simple scripts or one-off tooling."
---

# Python Design

Prefer designs that expose domain meaning, constrain invalid states, and keep side effects visible. Apply the following workflow proportionally: a small CRUD service rarely needs every pattern.

## Design workflow

1. **Identify invariants and lifecycle states.** State what must always be true and which transitions are valid before choosing classes or tables.
2. **Mark trust boundaries.** Distinguish untrusted HTTP, queue, file, and database data from values already accepted by the domain.
3. **Choose the smallest useful representation.** Use a primitive when it is unambiguous, `NewType` for static identity, a validated value object for runtime invariants, an enum for a closed set of constants, and a union of dataclasses for variants with different data.
4. **Introduce interfaces only at real substitution points.** Choose `Protocol` for structural compatibility and `ABC` for an explicit nominal hierarchy with shared behavior or runtime instantiation checks.
5. **Separate decisions from effects where it improves testing.** Pass time, randomness, and fetched data into deterministic logic; keep transactions, retries, logging, and I/O in an orchestration layer.
6. **Place invariants at the durable boundary.** Use application validation for useful errors and database constraints for facts that must survive every writer.
7. **Explain tradeoffs.** Recommend only the patterns whose benefit exceeds their ceremony, migration cost, and runtime overhead.

## Model variants explicitly

Use an explicit union for exhaustiveness. An `ABC` is extensible and does not make a hierarchy sealed.

```python
from dataclasses import dataclass
from datetime import datetime
from typing import NewType, TypeAlias, assert_never

CallId = NewType("CallId", str)

@dataclass(frozen=True)
class CallStarted:
    call_id: CallId
    started_at: datetime

@dataclass(frozen=True)
class CallEnded:
    call_id: CallId
    ended_at: datetime
    duration_s: int

CallEvent: TypeAlias = CallStarted | CallEnded

def handle(event: CallEvent) -> None:
    match event:
        case CallStarted(call_id=call_id):
            print("started", call_id)
        case CallEnded(call_id=call_id, duration_s=duration_s):
            print("ended", call_id, duration_s)
        case _ as unreachable:
            assert_never(unreachable)
```

Prefer frozen dataclasses for immutable values and events. Keep entities mutable when identity-preserving state transitions or an ORM require it; isolate and test those mutations.

Read [references/modeling-patterns.md](references/modeling-patterns.md) when choosing among unions, enums, lifecycle-specific types, `NewType`, and validated value objects.

## Parse untrusted data at boundaries

Parse boundary data into domain values before decision-heavy logic uses it. Use Pydantic when it fits the existing stack; do not add it solely to wrap trusted database rows or already typed internal values.

Treat `NewType` as a static-checking tool, not a validator: it returns the original value unchanged at runtime. Use a validated value object or boundary schema when a value must reject malformed input.

Primitive values remain appropriate when the domain does not require stronger identity or invariants. Avoid wrapping every `str` and `int` mechanically.

## Minimize accidental nullability

Before introducing `T | None` or a nullable column, decide in order:

1. If absence is a genuine domain state, model it explicitly with `None` or a named variant.
2. If absence represents a lifecycle stage, consider separate before/after types.
3. If absence means “not loaded,” change the query or projection type rather than weakening the domain model.
4. If callers must distinguish several failure reasons, use a result union instead of overloading `None`.

Keep a field non-nullable only when every valid instance has a value for it. Do not invent placeholder values to avoid `NULL`.

## Choose boundaries deliberately

- Use a concrete type until multiple implementations, testing seams, or dependency direction justify an interface.
- Use `Protocol` when consumers need a structural capability without forcing implementors to inherit or import the interface.
- Use `ABC` when implementations should join an explicit hierarchy, shared implementation matters, or preventing incomplete instances at runtime is useful.
- Verify Protocol compatibility where the value is assigned or passed. Expect an incomplete ABC subclass to fail when instantiated; do not describe this as enforcement at class definition.

## Handle failures at the right abstraction

Distinguish expected domain outcomes from programming and infrastructure failures:

- Use `T | None` for a single self-explanatory absence.
- Use a result union when callers need structured, exhaustive failure information.
- Raise a documented domain exception such as `ValueError` when invalid construction is exceptional for the API and callers naturally handle exceptions.
- Let programming errors propagate.
- Catch infrastructure exceptions where context can be added, a transaction can be rolled back, or retry/translation policy can be applied. Avoid both blanket catching and an artificial “catch exactly once” rule.

Read [references/boundary-and-error-patterns.md](references/boundary-and-error-patterns.md) when designing boundary parsing, interfaces, result types, exception translation, or transaction handling.

## Align application and database invariants

Use `NOT NULL`, `CHECK`, unique constraints, foreign keys, and transactions for durable relational invariants. Choose native enums only after considering the database dialect and migration frequency; a lookup table or checked text column may evolve more easily.

Do not assume application checks prevent races. Use database uniqueness, locking, isolation, or idempotency mechanisms when concurrent writers matter.

## Present a recommendation

When designing or reviewing code, provide:

1. Relevant invariants and assumptions
2. The smallest recommended design
3. Why each introduced abstraction earns its cost
4. Important alternatives and tradeoffs
5. Migration and testing implications

Avoid forcing domain-heavy architecture onto straightforward data plumbing.
