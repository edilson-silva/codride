---
description: Execute the next phase of plan.md for a work item
argument-hint: <work item folder>
---

# Engineer Work

We're currently working on the item specified in the following work item folder:

<folder>
#$ARGUMENTS
</folder>

The folder follows the `.claude/work/<type>/<slug>` convention (e.g. `.claude/work/fix/password-reset-plus-alias`), where `<type>` is a Conventional Commits type (`feat`, `fix`, `docs`, `chore`, `refactor`, ...).

To work on this, you should:

- Read every markdown file in the folder.
- Review `plan.md` and identify which phase is currently in progress.
- Present the user with a plan for tackling the next phase.

---

## GitHub issue tracking (mandatory)

**The development workflow must stay in sync with the GitHub issue in real time.**

### 1. When starting work on an issue

**Before writing any code:**

1. Identify the issue number (usually in the work item's slug, e.g. `fix/123-password-reset`, in `context.md`, or in the branch name matching that slug). If you can't find it, ask the user.
2. Move the issue to "in progress": `gh issue edit <number> --add-label "status:in-progress" --remove-label "status:todo"` (adjust label names to whatever the repo already uses — check with `gh label list` first).
3. Confirm the move in chat.

### 2. During development

- Keep the issue labeled as in-progress while you work.
- Comment on significant updates — blockers found, important technical decisions, progress on long phases:
  ```
  gh issue comment <number> --body "✅ Phase X complete - [summary of what was done]"
  ```

### 3. When opening a PR

1. Move the issue to review: `gh issue edit <number> --add-label "status:in-review" --remove-label "status:in-progress"`.
2. Comment the PR link on the issue: `gh issue comment <number> --body "PR opened for review: #<pr-number>"`.
3. Update `plan.md` with the PR link.

### 4. When done

After the PR merges:
1. Close the issue: `gh issue close <number> --comment "Shipped in #<pr-number>."`
2. If your workflow doesn't auto-close via a "Closes #<number>" line in the PR description, do it explicitly here.

---

Important: when implementing the current phase, use build/review/test sub-agents where appropriate to preserve as much of your own context as possible.

Every time you complete a phase of the plan:
- Pause and ask the user to validate your work.
- Make any changes needed until approved.
- Update the corresponding phase in `plan.md`, marking what was done and adding notes useful to whoever picks up the next phase — decisions made, open questions, etc.
- Only start the next phase once the user agrees you should. When you do, mark the new phase as in progress in `plan.md`.

## ADR-guided implementation (proactive, not reactive)

**If `context.md` contains a "Critical Project Rules" section:**

### 1. Before implementing each task

Read the critical rules relevant to the task **before** writing code. Plan the implementation around them:

```
Task: Create UserService

Applicable rules (from context.md):
- ADR-007: Services must use Repositories (not the ORM/DB client directly)
- ADR-003: Shared enums live in /shared/enums/
- ADR-012: Tests are mandatory

Implementation plan:
1. Create UserRepository first (prerequisite)
2. Create UserService, injecting the repository
3. If a new enum is needed, create it in /shared/enums/
4. Write tests alongside the implementation
```

Decide file locations and creation order (dependencies first) before creating anything.

### 2. While implementing

Follow the plan: create files in the right location, follow architectural patterns from the start, use the mandatory conventions. The goal is code that's compliant on the first pass, not code that gets refactored after a violation is found:

```
✅ Right (proactive): read ADR-007 → create the repository → create the service that
   injects it → compliant from the start.

❌ Wrong (reactive): create the service using the ORM directly → a checker flags the
   ADR-007 violation → refactor afterward → wasted time and tokens.
```

### 3. After implementing

Run a quick sanity check — this should confirm compliance, not hunt for problems:
- If `adr-compliance-checker` is available, invoke it.
- If it finds a violation, treat it as an exception/edge case worth discussing, not the expected outcome.

### 4. Report at the end of the phase

Include in `plan.md`'s notes for that phase which ADRs were applied and how (e.g. "ADR-007: Repository Pattern → AuthRepository created and injected").

### 5. When there are no ADRs

If `context.md` has no "Critical Project Rules" section: follow general good practices and the project's existing conventions, and skip this section entirely.

## Test-first implementation (TDD, red-green-refactor)

**If `context.md` or the source issue has acceptance criteria (`Given/When/Then`, from `/product:spec`):**

### 1. Before implementing each task

For each acceptance criterion the task covers, write the test that captures it first — and confirm it fails for the right reason (red) — before writing the implementation.

### 2. While implementing

Write just enough code to make that test pass (green), then refactor for quality once it's passing, without changing behavior. Repeat per criterion.

```
✅ Right (TDD): read the Given/When/Then for "reset works with a plus-aliased email" →
   write a failing test asserting it → implement until it passes → refactor.

❌ Wrong (reactive): implement the feature → run /engineer:coverage at the end →
   discover the plus-alias case was never tested → retrofit a test and hope the
   implementation is actually right.
```

### 3. After implementing

Run the test suite for the files you touched. Every test you wrote, and every existing one, should pass before moving to the next task.

### 4. Report at the end of the phase

Note in `plan.md` which acceptance criteria were covered by which tests.

### 5. When there's nothing to test-drive

Skip this section for work that isn't behavior to verify (a docs change, a config tweak, a pure refactor with no behavior change) — use judgment, don't force a red-green cycle where it doesn't fit.

---

## Core principles

> **ADRs are implementation guides, not error checkers.** Consult them as part of planning, implement in line with them the first time, and treat validation as confirmation — not correction.

> **Acceptance criteria are written before the code that satisfies them, not after.** A test that only ever existed to pass against code that already works isn't verifying the behavior — it's just restating it.

---

Now look at the current development phase and give the user a plan for how to approach it.
