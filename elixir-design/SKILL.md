---
name: elixir-design
description: "Design or review non-trivial Elixir domain models, module boundaries, and process structure. Use when choosing data representations, modeling lifecycle states or variants, defining behaviours or protocols, handling expected errors versus crashes, reducing nil, parsing untrusted input, separating pure logic from side effects, or deciding whether a concern justifies a process. Apply these patterns contextually; do not use for simple scripts or one-off mix tasks."
---

# Elixir Design

Prefer designs that expose domain meaning, constrain invalid states, and keep side effects visible. Apply the following workflow proportionally: a small CRUD context rarely needs every pattern.

## Design workflow

1. **Identify invariants and lifecycle states.** State what must always be true and which transitions are valid before choosing structs, modules, or storage.
2. **Mark trust boundaries.** Distinguish untrusted HTTP, message, file, and external-service data from terms the domain has already accepted.
3. **Choose the smallest useful representation.** Use a bare term when it is unambiguous, an atom for a closed set of constants, a struct when a group of fields travels together and deserves a name, a tagged tuple or struct union for variants carrying different data, and an opaque type when construction must be controlled.
4. **Introduce indirection only at real substitution points.** Choose a behaviour for a named contract implemented by several modules, a protocol for dispatch driven by the data's type, and a plain module otherwise.
5. **Separate decisions from effects.** Pass time, randomness, and fetched data into pure functions; keep I/O, retries, and logging in a thin shell around them.
6. **Decide which failures are values and which are crashes.** Expected outcomes become return values; programming errors crash and are handled by supervision.
7. **Reserve processes for runtime properties.** Concurrency, isolation, and state that outlives a call justify a process; code organization does not.
8. **Explain tradeoffs.** Recommend only the patterns whose benefit exceeds their ceremony, migration cost, and runtime overhead.

## Model variants explicitly

Give a union a name and match it without a catch-all clause.

```elixir
defmodule Call do
  defmodule Started do
    @enforce_keys [:call_id, :started_at]
    defstruct [:call_id, :started_at]
    @type t :: %__MODULE__{call_id: String.t(), started_at: DateTime.t()}
  end

  defmodule Ended do
    @enforce_keys [:call_id, :ended_at, :duration_s]
    defstruct [:call_id, :ended_at, :duration_s]
    @type t :: %__MODULE__{
            call_id: String.t(),
            ended_at: DateTime.t(),
            duration_s: non_neg_integer()
          }
  end

  @type event :: Started.t() | Ended.t()

  @spec describe(event()) :: String.t()
  def describe(%Started{call_id: id}), do: "started #{id}"
  def describe(%Ended{call_id: id, duration_s: seconds}), do: "ended #{id} after #{seconds}s"
end
```

Elixir has no `assert_never`. Exhaustiveness comes from omitting the catch-all: an unhandled variant raises `FunctionClauseError` or `CaseClauseError` at the site that failed to handle it. A `_ ->` or `_other ->` clause silently absorbs every variant added later, and is the main way exhaustiveness is lost. Add one only when the fallback is a genuine domain decision, not as defensive padding.

Read [references/modeling-patterns.md](references/modeling-patterns.md) when choosing among atoms, tagged tuples, struct unions, opaque types, and protocols, or when writing typespecs.

## Constrain construction

Terms are immutable, but a struct is not validated. Nothing stops another module from writing `%Money{amount: -1}` literally. Use `@enforce_keys` for fields with no meaningful default, and an `@opaque` type with a `new/1` constructor when a value must reject malformed input:

```elixir
defmodule Money do
  @enforce_keys [:cents]
  defstruct [:cents]

  @opaque t :: %__MODULE__{cents: non_neg_integer()}

  @spec new(integer()) :: {:ok, t()} | {:error, :negative_amount}
  def new(cents) when is_integer(cents) and cents >= 0, do: {:ok, %__MODULE__{cents: cents}}
  def new(_cents), do: {:error, :negative_amount}

  @spec to_cents(t()) :: non_neg_integer()
  def to_cents(%__MODULE__{cents: cents}), do: cents
end
```

Opacity is a Dialyzer-enforced convention, not a runtime guarantee. It works only if callers use the module's functions instead of matching its fields. Pattern matching another module's struct internals across a boundary couples to its shape rather than its contract; when that coupling spreads, apply the `coupling-cohesion` skill to the boundary rather than adding more accessors.

Use a plain struct without opacity when the fields are the contract and every combination of them is valid.

## Parse untrusted data at boundaries

Convert boundary data into domain terms once, at the edge. Threading raw maps with string keys through decision logic pushes shape-checking into every downstream function.

Use the validation library already in the project if one is present. Do not add one solely to wrap terms the domain has already accepted.

Convert incoming strings to atoms with `String.to_existing_atom/1`. `String.to_atom/1` on untrusted input grows the atom table without bound and eventually brings down the node.

## Minimize accidental nil

Before introducing a nilable field, decide in order:

1. If absence is a genuine domain state, model it explicitly with `nil` or a named variant.
2. If absence represents a lifecycle stage, use separate before/after structs.
3. If absence means "not loaded," change the query or the projection type rather than weakening the domain struct.
4. If callers must distinguish several failure reasons, return `{:error, reason}` instead of overloading `nil`.

Keep a field non-nilable only when every valid instance has a value for it. Do not invent placeholder values to avoid `nil`.

`nil` propagates quietly in Elixir: it flows through `Access` chains such as `user[:profile][:email]` and through pipelines until some unrelated function raises on it. The failure then surfaces far from the field that was actually absent, so prefer an explicit match or `Map.fetch/2` at the point where absence matters.

## Choose boundaries deliberately

- Use a plain module until multiple implementations, a testing seam, or dependency direction justifies indirection.
- Use a **behaviour** when several modules implement one named contract. `@callback` plus `@behaviour` gives compile-time warnings for missing callbacks, and the implementation can be swapped through configuration in tests.
- Use a **protocol** when dispatch is driven by the type of the data and consumers you do not control need to extend it, as with `Enumerable` or `Inspect`. Protocols dispatch at runtime and rely on consolidation for speed.
- The distinction is not stylistic: a behaviour describes what a **module** does; a protocol describes what can be done to a **value**. Choosing by taste rather than by that difference is the common error.

## Handle failures at the right abstraction

Distinguish expected domain outcomes from programming errors and infrastructure failures:

- Return `{:ok, value} | :error` for a single self-explanatory absence, matching `Map.fetch/2`.
- Return `{:ok, value} | {:error, reason}` when callers must distinguish reasons. Keep `reason` a matchable term such as an atom or tagged tuple, not a formatted string.
- Follow the `foo/1` and `foo!/1` convention: return a tuple when failure is an ordinary branch, raise when failure means the caller made a mistake.
- Let programming errors crash. Do not `rescue` a bug into an error tuple; that hides the stacktrace and moves the failure somewhere less diagnosable.
- Use `rescue` or `catch` where the layer can act: adding operation context, releasing a resource, or applying a retry or translation policy at a boundary.

Use `with` to chain fallible steps, and keep its `else` clauses specific. A single `else` clause that collapses every failure into one term discards exactly the information the error tuples carried.

Read [references/boundary-and-error-patterns.md](references/boundary-and-error-patterns.md) when designing boundary parsing, behaviours or protocols, error tuple contracts, `with` chains, or failure translation.

## Reserve processes for runtime concerns

A process is justified by a runtime property: state that must outlive a call, concurrency, isolation of failure, serialized access to a resource, or timed work. Encapsulation is not one of them — a module already provides that.

A GenServer wrapping otherwise pure functions adds a bottleneck, a mailbox, a supervision obligation, and a serialization point, while making the logic harder to test. Write the logic as pure functions in a plain module first, then add a process only when a runtime property demands it.

Read [references/process-and-otp-patterns.md](references/process-and-otp-patterns.md) when deciding whether a concern needs a process, structuring a functional core against an effectful shell, choosing among `Task`, `Agent`, and `GenServer`, or designing supervision and restart behaviour.

## Place invariants at the durable boundary

In-process checks do not prevent races between concurrent writers, and a check followed by a write is not atomic. Facts that must survive every writer belong in the durable store — uniqueness constraints, referential integrity, atomic transactions — or in an idempotency mechanism keyed by a client-supplied identifier.

Application-level validation earns its place by producing useful errors, not by providing guarantees. A GenServer serializing writes is a real guarantee only while it is the sole writer, and only within one node.

## Present a recommendation

When designing or reviewing code, provide:

1. Relevant invariants and assumptions
2. The smallest recommended design
3. Why each introduced abstraction earns its cost
4. Important alternatives and tradeoffs
5. Migration and testing implications

Avoid forcing domain-heavy architecture onto straightforward data plumbing.
