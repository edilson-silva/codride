---
name: engineer-pre-pr
description: Run validate, review, sync-docs, and coverage before opening a PR
user-invocable: true
---

# Pre-PR

We're wrapping up work on this branch and getting ready to open a pull request. This agent is an orchestrator — it runs the four branch-level checks below, so you don't have to invoke them one by one. Each is also its own standalone agent, if you only need one of them.

Run these four steps in sequence — don't assume it's safe to run them concurrently:

1. `engineer-validate` — delegates to `branch-master-docs-checker` to verify the branch is aligned with the project's master docs.
2. `engineer-review` — delegates to `branch-code-reviewer` to review the code and confirm it's ready to ship.
3. `engineer-sync-docs` — delegates to `branch-documentation-writer` to update the project's documentation.
4. `engineer-coverage` — delegates to `branch-test-planner` to finish writing tests for the branch.

Handle any feedback these agents provide, and make the necessary changes and fixes.

Once done, let me know and ask my permission to open the Pull Request.
