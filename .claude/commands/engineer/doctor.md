---
description: Pre-flight health check for a CoDriDe setup — what's configured, what's missing, what needs attention
---

# Doctor

Run this before diving into the pipeline on a project you just adopted CoDriDe into, or periodically on one you've been using it on for a while. It doesn't fix anything by itself — it reports, so you can decide what to act on.

## Checks

Run all of these and collect the results before reporting — don't stop at the first failure.

### 1. `gh` CLI
- `gh --version` — installed?
- `gh auth status` — authenticated? Which account?
- `git remote get-url origin` — does a remote exist, and is it actually a `github.com` URL? (If it's GitLab/Bitbucket/none, most of this framework's product-track and issue-sync commands won't work — say so plainly rather than letting them fail deep in a command later.)

### 2. Default branch
- `git remote show origin | sed -n '/HEAD branch/s/.*: //p'` (or `gh repo view --json defaultBranchRef -q .defaultBranchRef.name`).
- Report it explicitly — this is what `branch-code-reviewer`, `branch-documentation-writer`, `branch-test-planner`, and `branch-master-docs-checker` compare branches against. If it isn't `main`, that's fine, just confirm it resolved correctly rather than silently assuming.

### 3. Test suite
- Look for the project's test configuration: a `test` script in `package.json`, `pytest.ini`/`pyproject.toml`'s `[tool.pytest]`, `jest.config.*`, `vitest.config.*`, or equivalent.
- If nothing is found, say so explicitly — `/engineer:pr` assumes a test suite exists and can be run; if there isn't one, that assumption needs to be surfaced now, not discovered mid-PR.

### 4. Master docs status
- Does `docs/business-context/` exist? Does it have an `index.md`?
- Does `docs/technical-context/` exist? Which shape does it have — `project-briefing.md` (from `/engineer:discover`), `index.md` (from `/bootstrap:tech-docs`), neither, or (unusually) both?
- Does `docs/technical-context/adr/` (or `docs/adr/`, `adr/`) exist, and roughly how many ADRs are in it?

### 5. GitHub label taxonomy
- `gh label list` — check whether the labels this framework's commands reference by default exist: `status:backlog`, `status:todo`, `status:in-progress`, `status:in-review`, `priority:critical`, `priority:high`, `priority:medium`, `priority:low`, `bug`, `enhancement`, `improvement`, `research`. (`module:*` labels are created dynamically per module by `github-project-sync`, so there's nothing fixed to check there.)
- Missing labels aren't an error — every command that uses them already says "adjust to what the repo uses" — but report which ones are missing so the human can decide whether to create them or keep redirecting commands to different names each time.

### 6. Monorepo shape
- Check for `workspaces` in `package.json`, `pnpm-workspace.yaml`, `nx.json`, `turbo.json`, or `lerna.json`.
- If found, note it — `/engineer:discover` handles monorepos by analyzing each service separately; if `/engineer:discover` was already run before the project became a monorepo (or vice versa), the briefing may be stale in a specific way worth flagging.

### 7. CoDriDe framework files
- `.claude/agents/` — are the 8 core agents present and unmodified-looking (spot check `branch-code-reviewer.md` exists)? Any `project-*.md` agents already added?
- `.claude/work/` — any stale/abandoned work items sitting there from an interrupted session?

## Output

```markdown
# CoDriDe Doctor Report

## ✅ Working
- [Each check that passed, one line each]

## ⚠️ Missing (optional, but the pipeline is sharper with it)
- [Each gap that isn't blocking, with the one command that fixes it]

## ❌ Blocking (these will break specific commands)
- [Each real problem, which command(s) it breaks, and how to fix it]

## Summary
- Default branch: [detected]
- Test suite: [found / not found]
- Master docs: [business-context: yes/no, technical-context: shape or none]
- Monorepo: [yes, N services / no]
```

Don't fix anything automatically — this command's only job is to tell the human what's true about their setup right now.
