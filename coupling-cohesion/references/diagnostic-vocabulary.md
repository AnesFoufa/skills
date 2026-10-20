# Diagnostic vocabulary

Use these labels to compress an explanation after tracing the actual dependency. They are diagnostic vocabulary, not a maturity ladder or a reason to refactor by themselves.

## Cohesion

| Pattern | What holds the module together | Question it raises |
|---|---|---|
| Coincidental | Placement only, often a generic `utils` bucket | Which responsibilities are hiding here? |
| Logical | Similar operations selected by a control value | Is the caller steering internal behavior? |
| Temporal | Work performed at the same lifecycle moment | Does the lifecycle event itself form a responsibility? |
| Procedural | Required execution order | Who owns the workflow and its failure policy? |
| Communicational | Operations on the same data | Is the data a genuine domain concept or just shared storage? |
| Sequential | One operation's output feeds the next | Is this one transformation pipeline? |
| Functional | One well-defined responsibility | Does the interface expose that responsibility cleanly? |

Functional cohesion is often strong, but temporal, procedural, or communicational cohesion can be correct when it expresses a real lifecycle, workflow, or domain concept. A module may also contain several related operations without being incoherent.

## Coupling

| Pattern | Knowledge crossing the seam | Typical pressure |
|---|---|---|
| Content | Another module's private representation or behavior | Internal changes propagate directly |
| Common | Shared mutable or global state | Ownership and mutation order become ambiguous |
| External | A shared protocol, file format, schema, or device contract | Evolution is constrained by an outside authority |
| Control | A flag or command selects the callee's internal path | Caller learns the callee's decisions |
| Stamp | A composite value crosses the seam but only arbitrary pieces matter | Representation changes propagate unnecessarily |
| Data | Only contract-relevant values cross | Dependency is explicit and narrow |

Data coupling is not an automatic target. Passing a cohesive `Order` is clearer than exploding it into unrelated primitives when the receiver genuinely operates on an Order. External coupling may be unavoidable; isolate it where translation and compatibility policy belong.

## Common misdiagnoses

- **Large means incohesive.** Size is only a search hint. A large module can own one deep responsibility.
- **Many imports means highly coupled.** Count knowledge and change propagation, not import statements.
- **An interface lowers coupling.** An interface that mirrors one implementation adds a name, not independence.
- **Frequent co-change means merge.** Separate domain concepts can change together because one feature crosses them.
- **Pure functions deserve separate modules.** Extraction can reduce locality when callers still own all sequencing and invariants.
- **Few dependencies means good design.** A module can be isolated because it is useless, or hide coupling in globals, reflection, events, or string contracts.
