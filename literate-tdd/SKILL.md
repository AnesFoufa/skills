---
name: literate-tdd
description: "Develop a feature or bug fix using literate TDD: docs first, ephemeral worktree, e2e regression tracer, per-component red-green cycles, squash into one narrative commit. Use when user mentions 'literate TDD', 'worktree', 'docs first then tests', 'literate programming', or wants to develop a feature/fix with explicit methodology."
---

# Literate TDD

Docs-first, worktree-isolated, tracer-bullet TDD that squashes into one narrative commit.

## 1. Setup: ephemeral worktree

```bash
BASE=$(git rev-parse HEAD)          # save — needed for squash in step 6
BRANCH=feature/<TICKET>-<slug>
WORKTREE=~/<repo>-<slug>
git worktree add -b $BRANCH $WORKTREE HEAD
cd $WORKTREE
```

All work happens in the worktree. The main branch stays clean.

## 2. Docs first (literate step)

Before any test or code, update the relevant doc(s):

- **Algorithm/architecture doc** — add or amend the section describing the behavior you are about to implement or fix. State the invariant explicitly.
- **Data model doc** — update field tables, add the field to invariants.

Commit: `<TICKET>: document <behavior>`

Rule: if you cannot write the doc, you do not understand the problem well enough yet.

## 3. E2e regression test — RED

Write the end-to-end (or integration) test that **reproduces the failure**. Run it against real infrastructure (local Docker Compose is acceptable; pure mocks are not — they defeat the purpose of a tracer bullet).

- Run it. Confirm RED. Commit: `<TICKET>: e2e test for <behavior> (RED)`
- This commit is proof the bug existed / the feature was missing.

## 4. Per-component red-green cycles

Work bottom-up: core → service → adapter. For each component:

```
a. Write ONE unit test for that component's next behavior  →  confirm RED
b. Write the minimal code to pass — nothing speculative    →  confirm GREEN
c. Commit: <TICKET>: <fix description>
```

TDD rules that apply throughout:
- **One behavior per cycle.** Never write multiple tests before making them pass.
- **Test through the public interface only.** Tests that reach into internals break on refactor, not on behavior change.
- **Minimal code to pass.** No anticipation of future tests.
- **Never refactor while RED.** Get to GREEN first, then clean up.
- **Vertical slices, not horizontal.** `test1→impl1, test2→impl2` — not all tests then all impls.

## 5. Full suite sanity check

Run the complete unit test suite. No skips allowed. Fix any regressions before proceeding.

## 6. Squash + narrative commit

Use the `$BASE` saved in step 1:

```bash
git reset --soft $BASE
git commit -m "<TICKET>: <one-line summary>

## Problem
<what was wrong and why>

## Approach
<the design decision — what changed and why>

## Red -> Green sequence
1. docs: ...
2. e2e RED: ...
3. unit RED (<component>): ...
4. GREEN (<component>): ...
..."
```

## 7. Fast-forward main and clean up

If main moved forward since you branched, rebase first:

```bash
git rebase origin/main   # resolve any conflicts, then continue
```

Then merge and clean up:

```bash
cd <main-repo>
git merge --ff-only $BRANCH
git worktree remove $WORKTREE
git branch -d $BRANCH
```

## Checklist

```
[ ] Doc written before first test
[ ] E2e test confirmed RED before any code change
[ ] Each component: unit test RED then GREEN, one behavior at a time
[ ] Tests use public interfaces only
[ ] Full suite passes, no skips
[ ] Squash commit tells the red->green story
[ ] Worktree removed, ephemeral branch deleted
```
