# Boundary and Error Patterns

Use these patterns when data crosses a system boundary or failures cross an architectural boundary. Examples target Elixir 1.15 or newer.

## Parse boundary data

Convert accepted input into a domain struct once, and return a matchable reason on failure.

```elixir
defmodule CreateUser do
  @enforce_keys [:email, :role]
  defstruct [:email, :role]

  @type t :: %__MODULE__{email: String.t(), role: :admin | :member}
  @type error :: {:missing, :email | :role} | {:invalid, :email | :role}

  @spec parse(map()) :: {:ok, t()} | {:error, error()}
  def parse(%{} = payload) do
    with {:ok, email} <- fetch_email(payload),
         {:ok, role} <- fetch_role(payload) do
      {:ok, %__MODULE__{email: email, role: role}}
    end
  end

  defp fetch_email(payload) do
    case Map.fetch(payload, "email") do
      {:ok, email} when is_binary(email) ->
        if String.contains?(email, "@"), do: {:ok, email}, else: {:error, {:invalid, :email}}

      {:ok, _other} ->
        {:error, {:invalid, :email}}

      :error ->
        {:error, {:missing, :email}}
    end
  end

  defp fetch_role(payload) do
    case Map.fetch(payload, "role") do
      {:ok, role} when role in ["admin", "member"] -> {:ok, String.to_existing_atom(role)}
      {:ok, _other} -> {:error, {:invalid, :role}}
      :error -> {:error, {:missing, :role}}
    end
  end
end
```

Two things carry the design:

- The reason is a tagged tuple, so a caller can render a field-specific message or map to a status code without parsing text.
- The role is checked against a literal allow-list *before* conversion. `String.to_existing_atom/1` only guarantees the atom already exists somewhere in the system, which is weaker than membership in the intended set; `String.to_atom/1` on untrusted input grows the atom table without bound and eventually takes down the node.

A validation library already in the project may replace this parser. Re-validating trusted internal terms is worth it only when the added safety justifies the cost.

## Behaviour versus protocol

A behaviour names a contract that **modules** implement:

```elixir
defmodule Notifier do
  @callback deliver(to :: String.t(), body :: String.t()) :: :ok | {:error, term()}
end

defmodule Notifier.Log do
  @behaviour Notifier

  @impl Notifier
  def deliver(to, body) do
    IO.puts("to #{to}: #{body}")
    :ok
  end
end

defmodule Alerts do
  @spec raise_alert(module(), String.t()) :: :ok | {:error, term()}
  def raise_alert(notifier, message) do
    notifier.deliver("ops@example.com", message)
  end
end
```

Substitute it in tests by passing or configuring a different module. `@impl` makes the intent explicit and warns when a callback name drifts.

A protocol names something that can be done to a **value**, and stays open to implementations written elsewhere:

```elixir
defprotocol Describable do
  @spec describe(t()) :: String.t()
  def describe(value)
end

defmodule Invoice do
  defstruct [:number]
end

defimpl Describable, for: Invoice do
  def describe(%Invoice{number: number}), do: "invoice #{number}"
end
```

Choose by asking which one varies. If the caller picks an implementation, that is a behaviour. If the data determines the implementation and unknown consumers must be able to add their own, that is a protocol. Reaching for a protocol to get dependency injection produces runtime dispatch and a wrapper struct where a module argument would have been enough.

## Represent expected failures

Match the shape to what callers need:

```elixir
# One self-explanatory absence.
@spec fetch(String.t()) :: {:ok, term()} | :error

# Reasons callers must distinguish.
@spec charge(String.t(), pos_integer()) ::
        {:ok, String.t()} | {:error, :insufficient_funds | :card_expired | {:declined, String.t()}}
```

Keep reasons matchable. `{:error, "card expired for user 42"}` forces callers to inspect a string and breaks the moment the wording changes; `{:error, {:card_expired, user_id}}` does not.

Chain fallible steps with `with`, keeping `else` clauses specific:

```elixir
def checkout(payload) do
  with {:ok, request} <- CreateOrder.parse(payload),
       {:ok, order} <- Orders.place(request),
       {:ok, receipt} <- Payments.charge(order) do
    {:ok, receipt}
  else
    {:error, {:missing, field}} -> {:error, {:bad_request, field}}
    {:error, :out_of_stock} -> {:error, :conflict}
    {:error, reason} -> {:error, {:payment_failed, reason}}
  end
end
```

The counter-example destroys the contract the tuples established:

```elixir
else
  _ -> {:error, :failed}
end
```

Omitting `else` entirely is often better than collapsing: the non-matching value passes through unchanged, so a caller that already understands `{:error, :out_of_stock}` still receives it.

## Translate failures at the right layer

Catch a failure where the layer can take meaningful action:

- Add operation context before re-raising or returning, so the failure identifies what was being attempted.
- Release resources in `after`, which runs whether the body succeeded, raised, or exited.
- Retry only where idempotency and a retry policy are known. A retried non-idempotent write is a duplicate, not a recovery.
- Map failures to a transport outcome at the delivery boundary — status code, message acknowledgement, or job retry decision.

```elixir
def load_config(path) do
  case File.read(path) do
    {:ok, contents} -> {:ok, contents}
    {:error, reason} -> {:error, {:config_unreadable, path, reason}}
  end
end
```

Avoid logging and re-raising the same failure at several layers; the result is one incident reported as several. Avoid `rescue` around a programming error: converting a bug into `{:error, _}` discards the stacktrace and defers the crash to somewhere less informative. Let it crash and let supervision restart the process.
