# Process and OTP Patterns

Use these patterns when the design question is about processes, effects, or failure recovery rather than data shape. Examples target Elixir 1.15 or newer.

## Functional core, imperative shell

Keep the decision in a pure function and the effect outside it. Pass time, randomness, and fetched data in as arguments.

```elixir
defmodule Subscription do
  @type t :: %{expires_at: DateTime.t(), grace_days: non_neg_integer()}
  @type decision :: :active | :in_grace | :expired

  @spec status(t(), DateTime.t()) :: decision()
  def status(%{expires_at: expires_at, grace_days: grace_days}, now) do
    grace_end = DateTime.add(expires_at, grace_days * 86_400, :second)

    cond do
      DateTime.compare(now, expires_at) != :gt -> :active
      DateTime.compare(now, grace_end) != :gt -> :in_grace
      true -> :expired
    end
  end
end

defmodule Subscriptions do
  @spec sweep(Subscription.t()) :: :ok
  def sweep(subscription) do
    case Subscription.status(subscription, DateTime.utc_now()) do
      :active -> :ok
      :in_grace -> Mailer.warn(subscription)
      :expired -> Billing.revoke(subscription)
    end
  end
end
```

`status/2` is tested with a table of timestamps and no setup. Only `sweep/1` needs stubs, and it contains no branching logic worth exhaustive testing. A pure function that returns a *description* of the effect — `{:send_warning, id}` rather than sending it — pushes the boundary further out when the decision is complex enough to justify it.

## When a process is justified

Add a process for a runtime property:

- State must outlive a single call and be reachable by more than one caller.
- Work should proceed concurrently with other work.
- A failure must be isolated so it does not take down the caller.
- Access to a resource must be serialized — a device, a connection, a rate limit.
- Work is timed, periodic, or must continue after the requester has gone.

Do not add a process for encapsulation, namespacing, or "so it has a lifecycle." A module already groups functions and hides internals. A GenServer that only calls pure functions on the state it was given costs a mailbox, a serialization point, a supervision obligation, and a test setup, and buys nothing.

The tell is a GenServer whose every `handle_call` is a one-line delegation and whose state never changes between calls. That is a module written the long way.

## Error kernel and let-it-crash

Split the system into a small kernel whose failure is unacceptable and a larger periphery whose failure is recoverable by restart.

- In the periphery, let processes crash. The stacktrace reaches the log, the supervisor restarts from a known-good state, and no partially-updated state survives.
- In the kernel, handle errors deliberately: validate before committing, keep the amount of irreplaceable state small, and persist what cannot be reconstructed.
- Do not `rescue` a programming error to keep a process alive. A process running on state that a bug already corrupted causes damage that is harder to trace than a restart.
- Handle expected outcomes as values, not crashes. A missing record is `{:error, :not_found}`; an unreachable database is a crash the supervisor and backoff handle.

Restart strategies encode dependencies between children, so choose one as a design statement:

- `:one_for_one` — children are independent.
- `:one_for_all` — children share state or a connection and are only valid together.
- `:rest_for_one` — later children depend on earlier ones.

If restarting one child leaves siblings holding a stale reference to it, the strategy is wrong, not the child.

## Task, Agent, GenServer, and supervised alternatives

Pick the lightest construct that expresses the requirement:

- **`Task`** — one unit of concurrent work with a result. `Task.async/await` for a bounded fan-out, `Task.async_stream/3` with `max_concurrency` for a collection, `Task.Supervisor` for work that must not crash its caller.
- **`Agent`** — state with no logic of its own. Simple, but every caller supplies the update function, which scatters the rules for valid state across call sites. Prefer a GenServer once invariants matter.
- **`GenServer`** — state plus behaviour, timed work, or serialized access.
- **`Registry`** — naming many dynamic processes without inventing an atom per process.
- **`DynamicSupervisor`** — one supervised process per session, connection, or entity.
- **`:ets`** — shared reads at high volume where a single process would be the bottleneck. It trades the serialization a GenServer provides for concurrency, so the invariants must tolerate that.

An unsupervised `spawn` or bare `Task.start/1` is a leak: nothing restarts it, nothing shuts it down cleanly, and its crash may go unreported. Start processes under a supervisor.

## State design inside a process

Give the state a struct and a type, so the shape is documented and Dialyzer can check it:

```elixir
defmodule Throttle do
  use GenServer

  defmodule State do
    @enforce_keys [:limit, :used]
    defstruct [:limit, :used]
    @type t :: %__MODULE__{limit: pos_integer(), used: non_neg_integer()}
  end

  @spec start_link(keyword()) :: GenServer.on_start()
  def start_link(opts), do: GenServer.start_link(__MODULE__, opts, name: __MODULE__)

  @spec take() :: :ok | {:error, :rate_limited}
  def take, do: GenServer.call(__MODULE__, :take)

  @impl GenServer
  def init(opts) do
    {:ok, %State{limit: Keyword.fetch!(opts, :limit), used: 0}}
  end

  @impl GenServer
  def handle_call(:take, _from, state) do
    case consume(state) do
      {:ok, state} -> {:reply, :ok, state}
      {:error, reason} -> {:reply, {:error, reason}, state}
    end
  end

  @spec consume(State.t()) :: {:ok, State.t()} | {:error, :rate_limited}
  def consume(%State{used: used, limit: limit} = state) when used < limit do
    {:ok, %State{state | used: used + 1}}
  end

  def consume(%State{}), do: {:error, :rate_limited}
end
```

The rules live in `consume/1`, which is tested directly without starting the process. `handle_call/3` only translates between the process protocol and that function.

Two further habits:

- Prefer `call` over `cast` when the caller needs to know the operation happened. `cast` returns `:ok` whether or not the process handled the message, or even exists, so a failure becomes invisible.
- Keep the client API in the same module as the callbacks, and keep `GenServer.call/2` out of other modules. Callers should not need to know that a function is backed by a process.
