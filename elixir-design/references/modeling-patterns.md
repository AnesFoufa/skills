# Modeling Patterns

Use these examples when the representation choice is the main design question. Examples target Elixir 1.15 or newer.

## Structs, opaque types, and smart constructors

Use `@enforce_keys` for fields that have no meaningful default. It raises at construction when a required field is missing, which catches the mistake at the site that made it.

```elixir
defmodule Reservation do
  @enforce_keys [:id, :seats]
  defstruct [:id, :seats, notes: nil]

  @type t :: %__MODULE__{id: String.t(), seats: pos_integer(), notes: String.t() | nil}
end
```

Add an `@opaque` type and a constructor when construction must enforce an invariant:

```elixir
defmodule Email do
  @enforce_keys [:value]
  defstruct [:value]

  @opaque t :: %__MODULE__{value: String.t()}

  @spec new(String.t()) :: {:ok, t()} | {:error, :invalid_email}
  def new(value) when is_binary(value) do
    if String.contains?(value, "@"), do: {:ok, %__MODULE__{value: value}}, else: {:error, :invalid_email}
  end

  @spec new!(String.t()) :: t()
  def new!(value) do
    case new(value) do
      {:ok, email} -> email
      {:error, :invalid_email} -> raise ArgumentError, "invalid email: #{inspect(value)}"
    end
  end

  @spec value(t()) :: String.t()
  def value(%__MODULE__{value: value}), do: value
end
```

Return a tuple from `new/1` when invalid input is an ordinary branch callers should handle. Provide `new!/1` for callers holding a literal or already-validated value, where failure means a bug.

A bare binary or integer stays appropriate when the domain needs no stronger identity or invariant. Wrapping every value mechanically adds ceremony without preventing a real mistake.

## Model lifecycle states without partial structs

Split types when fields become mandatory after a transition.

```elixir
defmodule DraftReport do
  @enforce_keys [:body]
  defstruct [:body]
  @type t :: %__MODULE__{body: String.t()}
end

defmodule PublishedReport do
  @enforce_keys [:body, :published_at]
  defstruct [:body, :published_at]
  @type t :: %__MODULE__{body: String.t(), published_at: DateTime.t()}
end

defmodule Reports do
  @spec publish(DraftReport.t(), DateTime.t()) :: PublishedReport.t()
  def publish(%DraftReport{body: body}, now) do
    %PublishedReport{body: body, published_at: now}
  end
end
```

`publish/2` cannot be handed an already-published report, and no function needs to check whether `published_at` happens to be set.

Keep one struct with a nilable field when absence is a stable, meaningful state and splitting the type would make consumers harder to follow.

## Select among atom, tagged tuple, struct union, and protocol

- **Atom** — a closed set of named constants whose members carry no data: `:pending | :active | :closed`. Cheap to match, cheap to serialize.
- **Tagged tuple** — lightweight variants carrying a small amount of positional data: `{:ok, value} | {:error, reason}`, `{:retry_after, seconds}`. Best when the shape is obvious from the tag.
- **Struct union** — variants carrying several named fields that consumers match on individually. Named fields survive refactoring better than tuple positions.
- **Protocol** — only when consumers outside the defining code must add their own implementations. A protocol is an open extension point; a union deliberately is not.

Declare the union once so the intended set is documented and Dialyzer can use it:

```elixir
@type status :: :pending | :active | :closed
@type outcome :: {:ok, status()} | {:error, :not_found | :forbidden}
```

## Typespecs and Dialyzer

Write `@spec` for public functions and `@type` for any shape named more than once. Use `@opaque` when callers must go through the module's functions, and `@typep` for types used only inside the module.

Dialyzer uses success typing: it reports what it can prove impossible, not everything that disagrees with the spec. Practical consequences:

- It catches calls that can never succeed, unreachable clauses, and specs contradicted by the code.
- It does not verify that a `case` covers every member of a declared union, so omitting the catch-all clause remains the mechanism that surfaces an unhandled variant.
- It does not check a spec that is merely looser than the implementation, so an over-broad spec such as `any()` silently disables the check.

Elixir's set-theoretic type system is progressively moving these checks into the compiler, so recent versions warn at compile time about some mismatches Dialyzer used to find later. It narrows what a separate Dialyzer run adds; it does not change the modeling advice here.

## Immutability is given; discipline is not

Terms cannot be mutated, but a struct threaded through a long pipeline can still drift out of its invariants when each step rebuilds it with `%{report | ...}`. Immutability guarantees no one else changed the value; it does not guarantee the value still makes sense.

Keep transformations in named functions that restore the invariant before returning, and keep the update syntax inside the module that owns the struct. A pipeline of raw `%{s | field: ...}` updates written by callers is the struct's invariants leaking into every call site.
