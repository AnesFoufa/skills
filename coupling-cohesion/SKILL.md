---
name: coupling-cohesion
description: "Evaluate and improve module boundaries through coupling and cohesion. Use when creating, splitting, merging, or reorganizing modules; extracting or moving responsibilities; introducing cross-module dependencies; changing public interfaces; reviewing architecture; or when the user mentions coupling, cohesion, module boundaries, god classes, or spaghetti code. Do not use for routine local edits that leave responsibilities and boundaries unchanged."
---

# Coupling and Cohesion

Keep responsibilities focused and dependencies necessary, explicit, and directed toward stable public contracts. Treat the classifications below as diagnostic vocabulary, not pass/fail rankings.

## Evaluate cohesion

Ask whether a module's elements contribute to a clear responsibility and usually change for the same reason. Treat difficulty naming that responsibility without "and" as a smell, not proof of poor cohesion.

Look for:

- Unrelated domain imports or reasons to change
- Functions that share neither data nor purpose
- Groups that can be understood, tested, and changed independently

Use these patterns to describe what you find:

```text
Coincidental    — elements grouped arbitrarily (for example, a generic "utils" module)
Logical         — similar operations selected by a caller-provided control value
Temporal        — operations grouped because they run at the same time
Procedural      — operations grouped because they execute in sequence
Communicational — operations centered on the same data
Sequential      — one operation's output feeds the next
Functional      — elements contribute to one well-defined responsibility
```

Prefer functional cohesion when it fits. Accept temporal, procedural, or communicational cohesion when it expresses a real architectural responsibility. Recommend a split only when the resulting modules have clearer responsibilities and are independently useful.

## Evaluate coupling

Ask how much one module must know about another, whether that knowledge is necessary, and whether the dependency points toward a stable abstraction or volatile implementation detail.

Look for:

- Imports from another module's private implementation
- Shared mutable or global state
- Control flags that expose the callee's internal decisions to callers
- Composite values whose arbitrary internal fields become part of the dependency
- Changes to an implementation detail that force unrelated consumers to change

Use these patterns to describe what you find:

```text
Content  — one module depends on or modifies another's internals
Common   — modules share global data
External — modules depend on the same external format, protocol, or contract
Control  — a caller passes instructions that select the callee's internal behavior
Stamp    — a callee accepts a structure but depends on only an arbitrary subset
Data     — a callee receives only the data its contract requires
```

Prefer narrow dependencies on useful, stable public contracts. Use an existing contract when it creates a meaningful boundary; do not invent an interface solely to make coupling appear lower. Pass a cohesive domain object when the callee genuinely operates on that concept; otherwise pass only the required data. Do not replace meaningful domain types with unrelated primitives merely to reduce the apparent amount of coupling.

## Decide whether to split or merge

Treat a module boundary as a design tool, not a goal. Make each boundary earn its coordination cost.

Consider splitting when one module contains responsibilities that change independently and the resulting modules would be meaningful on their own.

Consider merging when modules change for the same reason, cannot provide meaningful behavior independently, and their boundary creates more coordination than isolation. Look for bidirectional dependencies, forwarding-only layers, invariants or transactions that continually span the boundary, and tests that cannot exercise either module meaningfully in isolation. Treat frequent co-change as supporting evidence only: distinct domain concepts can participate in the same features while still deserving separate boundaries.

Keep modules separate when the boundary protects a real domain distinction, stable policy, ownership, security, deployment, or substitution point. Do not merge merely because modules collaborate frequently. Merge only when the combined module retains a clear responsibility and removes more accidental coupling than it introduces.

For example, merge a pricing module with a pricing service that only forwards every operation and has no independent policy or runtime boundary. Split pricing from notification delivery when they change for different reasons and can be understood and tested independently.

## Review the change

1. State the module's primary responsibility and reasons to change.
2. Identify the relevant callers, dependencies, and public contracts.
3. Compare the design before and after the proposed change.
4. Distinguish necessary domain coupling from accidental implementation coupling.
5. Recommend restructuring only when it materially improves understanding, testability, or changeability without adding unjustified indirection.
6. Report material deterioration with evidence and a concrete alternative. Do not flag harmless or unavoidable coupling merely because a dependency was added.

## Quick checks

```text
[ ] Can I state the module's responsibility and reason to change clearly?
[ ] Does this change introduce a responsibility that can vary independently?
[ ] Is each dependency necessary, explicit, and directed toward a useful stable contract?
[ ] Does either module depend on the other's private representation or decisions?
[ ] Is a passed object a genuine domain concept or merely a convenient bag of data?
[ ] Are neighboring modules independently meaningful, or is one mostly forwarding?
[ ] Does their boundary protect a real domain, policy, ownership, or runtime distinction?
[ ] Would splitting or merging improve changeability without adding unjustified indirection?
```
