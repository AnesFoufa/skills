---
name: coupling-cohesion
description: "Evaluate and improve coupling and cohesion when modifying a module. Use when modifying any module, adding a dependency, extracting code, or when user mentions 'coupling', 'cohesion', 'module boundary', 'god class', 'spaghetti', or 'refactor'."
---

# Coupling & Cohesion

Before modifying a module, assess its current state. Before finishing, verify you haven't made it worse.

## Cohesion — how strongly are the elements of this module associated?

Constantine defines cohesion as the strength of association among elements within a module. A practical signal: the module's purpose should be describable in a single noun or verb phrase. If you need "and", cohesion is low.

**Signs of low cohesion:**
- The module imports from many unrelated domains
- Functions inside it don't share data or purpose
- You can split it into two modules and neither needs the other

**Cohesion levels (worst → best):**
```
Coincidental    — elements grouped arbitrarily ("utils.py")
Logical         — elements perform logically similar functions, selected by a control flag passed by the caller ("handle_io(type)" that branches on type)
Temporal        — elements run at the same time ("startup routines")
Procedural      — elements follow a specific execution sequence but don't share data ("open, read, close")
Communicational — elements operate on the same data
Sequential      — output of one feeds the next
Functional      — elements all contribute to a single well-defined task  ← target
```

**Rule:** aim for functional cohesion. Anything below communicational (coincidental, logical, temporal, procedural) is poor cohesion — flag it and suggest a split or reorganisation.

## Coupling — how much does this module know about others?

Low coupling means a module depends on as little of another module as possible — ideally only its public interface, not its internals or structure.

**Signs of high coupling:**
- Changing module A requires changing module B
- A imports deeply into B's internals (`from b.internal.helpers import _fmt`)
- A passes large objects to B when B only needs one field
- A and B share a global data area

**Coupling levels (worst → best):**
```
Content   — A references B's interior: modifies its data, branches into it, or depends on its internal structure
Common    — A and B share access to a global data area
External  — A and B are coupled through an external format, protocol, or third-party contract
Control   — A passes a flag that tells B what to do
Stamp     — A passes a composite data structure; B uses only part of it
Data      — A passes only the data B needs           ← target
```

**Rule:** prefer data coupling. Pass only what the callee needs. Never reach into another module's internals.

## When modifying a module

1. **Assess before touching** — what is the current cohesion level? What is the coupling to its callers and dependencies?
2. **Does your change lower cohesion?** If you're adding something unrelated to the module's purpose, it belongs elsewhere.
3. **Does your change increase coupling?** If you're importing a new module or passing a richer object, look for a thinner interface.
4. **Flag to the user** any change that would worsen either metric, with a concrete alternative.

## Quick checks

```
[ ] Can I name this module's purpose without using "and"?
[ ] Does my change fit that purpose?
[ ] Am I importing a new module? Is there a narrower dependency?
[ ] Am I passing a large object? Can I pass only the needed fields?
[ ] Am I reading another module's internals? Should I go through its public interface?
[ ] Would this change force edits in other modules?
```
