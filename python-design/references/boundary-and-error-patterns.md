# Boundary and Error Patterns

Use these patterns when data crosses a system boundary or failures cross an architectural boundary. Examples target Python 3.11 or newer.

## Parse boundary data

Use the validation library already present in the project. Convert accepted input to domain values once rather than passing raw dictionaries through decision-heavy code.

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class CreateUser:
    email: str

def parse_create_user(payload: object) -> CreateUser:
    if not isinstance(payload, dict):
        raise ValueError("payload must be an object")
    email = payload.get("email")
    if not isinstance(email, str) or "@" not in email:
        raise ValueError("email must be valid")
    return CreateUser(email=email)
```

Pydantic, attrs, marshmallow, or framework-native schemas may replace the hand-written parser. Validate trusted internal data again only when the added safety justifies the runtime cost.

## Choose Protocol or ABC

Use a Protocol for structural dependency inversion:

```python
from typing import Protocol

class UserWriter(Protocol):
    def save_email(self, email: str) -> None: ...

def register(email: str, writer: UserWriter) -> None:
    writer.save_email(email)
```

Use an ABC for an explicit nominal family or shared implementation. `@abstractmethod` prevents instantiation until required methods are implemented; it does not seal the hierarchy.

## Represent expected failures

Use a small result union when callers need exhaustive failure information.

```python
from dataclasses import dataclass
from typing import Generic, TypeAlias, TypeVar, assert_never

T = TypeVar("T")
E = TypeVar("E")

@dataclass(frozen=True)
class Ok(Generic[T]):
    value: T

@dataclass(frozen=True)
class Err(Generic[E]):
    error: E

@dataclass(frozen=True)
class EmptyName:
    pass

ParseNameResult: TypeAlias = Ok[str] | Err[EmptyName]

def parse_name(raw: str) -> ParseNameResult:
    if not raw.strip():
        return Err(EmptyName())
    return Ok(raw.strip())

def display(result: ParseNameResult) -> str:
    match result:
        case Ok(value=name):
            return name
        case Err(error=EmptyName()):
            return "missing name"
        case _ as unreachable:
            assert_never(unreachable)
```

Do not introduce a generic result abstraction for every operation. `None`, a documented exception, or a domain-specific union may communicate the contract more clearly.

## Translate infrastructure failures

Catch an infrastructure exception where the layer can take meaningful action:

- Roll back or close a transaction at the transaction boundary.
- Add operation context before re-raising or translating an adapter exception.
- Retry only where idempotency and retry policy are known.
- Map failures to HTTP or worker outcomes at the delivery boundary.

Avoid logging and re-raising the same failure at several layers. Avoid catching `Exception` unless the outer process boundary must prevent one operation from terminating the process and preserves diagnostics.
