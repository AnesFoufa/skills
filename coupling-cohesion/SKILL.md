---
name: coupling-cohesion
description: "Diagnose and redesign module seams by tracing change pressure, responsibility, and leaked knowledge. Use when splitting, merging, extracting, or reorganizing modules; changing public interfaces or cross-module dependencies; reviewing architecture; or investigating god classes and spaghetti code. Skip routine local edits whose responsibilities and seams stay unchanged."
---

# Coupling and Cohesion

Find the **change gravity**: behavior that must move together belongs together; knowledge that crosses a seam must earn the coordination it creates.

Treat a module as any architectural unit with an interface and implementation—a class, file, package, subsystem, or process. Do not optimize for the fewest dependencies or the smallest files. Optimize for a design in which a likely change is local, invariants have one home, and callers know only what their work requires.

## Choose the depth of review

- **Change guardrail** — while implementing a scoped change, inspect only its boundary delta. Continue silently when responsibility and coupling stay sound; report a concern only when the change materially worsens them.
- **Focused seam review** — when the user names modules or a design decision, trace that seam and return a boundary brief.
- **Landscape review** — when the user asks for architectural candidates across a repository or subsystem, locate change-pressure hotspots and compare several seams. Read [references/visual-report.md](references/visual-report.md) and present the findings as its visual report.

Use [references/diagnostic-vocabulary.md](references/diagnostic-vocabulary.md) only when classical coupling or cohesion labels will sharpen the explanation. Evidence and consequences matter more than assigning a category.

## Trace change pressure

Start from a concrete behavior or plausible change, not the directory tree.

1. **Name the promise.** Complete: “This module owns ___.” Then list the distinct reasons it changes. An “and” is a lead to investigate, not an automatic split.
2. **Draw the seam.** Identify callers, public entry points, dependencies, data crossing the seam, shared state, errors, transactions, and tests. Read domain glossaries and relevant ADRs when present.
3. **Follow one change.** Ask which files, interfaces, and tests move when a representative rule changes. For a broad review, use recent history to find frequently changed areas, then inspect the code; co-change is supporting evidence, never the verdict.
4. **Mark leaked knowledge.** Record every place one side must know the other side's representation, call order, control flow, partial object shape, failure mechanics, or mutation timing.
5. **Separate necessary coupling from accidental coupling.** Shared domain meaning, atomic invariants, protocols, ownership, and runtime constraints may require coupling. Convenience imports, forwarding modules, control flags, and duplicated orchestration usually do not.

The trace is complete when every material arrow across the seam has a purpose and a change consequence.

## Put the seam on trial

Test the current design and its smallest credible alternative. Do not prescribe a split merely because a module is large, or an interface merely because a dependency exists.

### Split test

A split earns its seam when both sides:

- have names based on responsibilities rather than mechanics;
- can change, be understood, and be tested meaningfully on their own;
- communicate through a smaller contract than the implementation they hide; and
- do not force one invariant, transaction, or workflow to shuttle back and forth.

If the proposed halves continually coordinate, the split distributed one responsibility instead of discovering two.

### Merge test

Imagine deleting the seam. Merge or absorb a module when deletion concentrates the behavior and removes forwarding, call-order, or shared-invariant knowledge. Keep the seam when deletion erases a real domain, policy, ownership, security, deployment, or substitution distinction.

### Contract test

For each value or instruction crossing the seam, ask:

- Does the receiver need the whole concept, or merely fields from a convenient bag?
- Does a flag make the caller choose the receiver's internal path?
- Are callers coupled to construction, sequencing, retries, or exception details?
- Would narrowing the contract clarify the domain, or merely replace a meaningful type with primitives?
- Is there a real alternate implementation or dependency-direction need, or would a new interface be speculative ceremony?

## Choose the smallest structural move

Give one primary verdict:

- **Keep** — the seam protects a meaningful distinction and its coupling is necessary.
- **Move** — one responsibility is simply on the wrong side.
- **Narrow** — retain the modules but remove leaked knowledge from the contract.
- **Invert** — dependency direction conflicts with policy, ownership, or volatility.
- **Split** — independent responsibilities are trapped together.
- **Merge** — one responsibility is fragmented across a tax-producing seam.

Prefer moving behavior or reshaping an existing contract before inventing a new abstraction. Preserve user-selected frameworks, deployment boundaries, and recorded architecture decisions unless the evidence justifies explicitly reopening them.

## Prove the recommendation

Run a **change simulation** before recommending a structural move:

1. Pick a realistic next change.
2. List the modules, contracts, and tests it touches now.
3. List what it would touch after the proposed move.
4. Check where the invariant, transaction, and error policy live afterward.
5. Reject the proposal if it only relocates complexity, hides necessary coupling, or adds coordination without reducing leaked knowledge.

When implementation is requested, preserve behavior with tests through public interfaces and make only the chosen structural move. Verify that dependency direction and affected tests match the proposed after-state.

## Deliver the verdict

For a focused review, return a compact **boundary brief**:

1. **Verdict** — one of Keep, Move, Narrow, Invert, Split, or Merge, with confidence.
2. **Module promises** — one responsibility sentence per module.
3. **Pressure map** — a small before/after diagram when relationships are otherwise hard to see.
4. **Coupling ledger** — for each material dependency: knowledge crossing the seam, why it exists, and the change it propagates.
5. **Recommendation** — the smallest concrete move, naming affected modules and contracts.
6. **Change simulation** — before versus after touch points and test surface.
7. **Tradeoff** — what becomes worse or remains necessarily coupled.

Use strength labels rather than fake scores: **Strong**, **Worth exploring**, or **Speculative**. Distinguish observed facts from inferences. If the evidence supports keeping the current seam, say so plainly; “no refactor” is a successful review.
