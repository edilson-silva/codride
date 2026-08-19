---
description: Run validate, review, sync-docs, and coverage before opening a PR
---

# Pre-PR

We're wrapping up work on this branch and getting ready to open a pull request. This command is an orchestrator — it runs the four branch-level checks below, so you don't have to invoke them one by one. Each is also its own standalone command, if you only need one of them.

**Step 1 — run in parallel** (both are read-only, neither depends on the other's output — invoke both agents in a single message so they run concurrently):
- `/engineer:validate` — invokes `branch-master-docs-checker` to verify the branch is aligned with the project's master docs.
- `/engineer:review` — invokes `branch-code-reviewer` to review the code and confirm it's ready to ship.

Wait for both to finish before moving on.

**Step 2 — run sequentially** (both write to the branch, so avoid running them concurrently with each other or with anything from Step 1):
1. `/engineer:sync-docs` — invokes `branch-documentation-writer` to update the project's documentation.
2. `/engineer:coverage` — invokes `branch-test-planner` to finish writing tests for the branch.

Handle any feedback these agents provide, and make the necessary changes and fixes.

Once done, let me know and ask my permission to open the Pull Request.
