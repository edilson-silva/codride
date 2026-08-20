# CoDriDe

CoDriDe is a Context Driven Development (CDD) framework for Claude Code: a set of commands and agents (under `.claude/`) meant to be copied into another project's repo to structure how features get discovered, specified, built, and shipped with Claude.

The framework treats **master docs** as the project's "DNA": living documents (business context + technical context) that every feature gets checked against, both before it's built (`/product:validate`) and after (`branch-master-docs-checker`).

Project management runs through **GitHub Issues**, via the `gh` CLI.

For full command-by-command documentation, best practices, and adoption instructions, see [README.md](README.md).

## Pipeline

```
/warm-up
   │
   ├─ (new or unfamiliar project) /engineer:doctor ─── pre-flight check: gh auth, default branch,
   │                                                     test suite, master docs, labels, monorepo shape
   │
   ├─ (once, optional) /engineer:discover ─── scans ADRs + backend, writes docs/technical-context/project-briefing.md,
   │                                            checks existing code against those ADRs (adr-compliance-checker)
   │
   ├─ Product track
   │     /product:collect      ─ capture a raw idea/bug as a GitHub issue
   │     /product:refine       ─ turn it into a WHY/WHAT/HOW requirement doc
   │     /product:spec         ─ expand into a full PRD; writes docs/business-context/features/<slug>.md
   │     /product:validate     ─ validate against master docs
   │     /product:brainstorm   ─ structured exploration for open-ended decisions
   │     /product:sync-github  ─ sync docs/business-context/features/*.md (from spec/quick-spec) with GitHub Issues
   │     /product:quick-spec   ─ create a fully-specced issue directly from a task description; also writes features/<slug>.md
   │
   └─ Engineering track
         /engineer:context      ─ interview → context.md for one unit of work
         /engineer:architecture ─ design → architecture.md, cross-checked against context.md
         /engineer:plan         ─ phased plan.md (2-hour-sized phases)
         /engineer:work         ─ execute a phase, ADR-guided, syncs the GitHub issue
         /engineer:pre-pr       ─ orchestrator: runs the 4 below
              /engineer:validate   ─┐ branch vs. master docs   (parallel, read-only)
              /engineer:review     ─┘ code review
              /engineer:sync-docs  ─  update project docs      (sequential, writes)
              /engineer:coverage   ─  missing test coverage
         /engineer:pr           ─ commit, open PR, move the issue, handle review-bot comments
         /engineer:bump         ─ semver bump (pyproject.toml or package.json)
         /engineer:adr          ─ draft a new Architecture Decision Record
```

Each of `/engineer:validate`, `/engineer:review`, `/engineer:sync-docs`, and `/engineer:coverage` also runs standalone — use `/engineer:pre-pr` for the full sweep before a PR, or run just the one you need. Inside `/engineer:pre-pr`, `validate` and `review` run in parallel (both read-only, no shared risk); `sync-docs` and `coverage` run after, sequentially (both write to the branch).

`/engineer:context` and `/engineer:architecture` aren't just for features — run them for a fix, a chore, or anything substantial enough to benefit from a written plan, even before a full product-track spec exists.

Three practices are woven into the pipeline rather than bolted on as separate commands: `/product:spec` writes BDD acceptance criteria (`Given/When/Then`) per user story; `/engineer:work` implements test-first against them (red-green-refactor); `/engineer:architecture` has an optional DDD domain-model section for features where the domain is complex enough to warrant one. See [README.md](README.md) for the optional MCPs (Context7, a Playwright MCP) that pair with these.

## Example: one feature, end to end

```
/product:collect "customers want to export their order history as CSV"
# → creates GitHub issue #42

/product:refine 42
# → issue #42 updated with WHY/WHAT/HOW

/product:validate "CSV export of order history, issue #42"
# → confirms it doesn't violate any master doc

/product:spec 42
# → issue #42 becomes a full PRD

/engineer:context feat/csv-order-export
# → .claude/work/feat/csv-order-export/context.md

/engineer:architecture feat/csv-order-export
# → .claude/work/feat/csv-order-export/architecture.md, cross-checked against context.md

/engineer:plan feat/csv-order-export
# → .claude/work/feat/csv-order-export/plan.md (phased)

/engineer:work .claude/work/feat/csv-order-export
# → implements Phase 1, syncs issue #42 to "in progress", repeat per phase

/engineer:pre-pr
# → /engineer:validate + /engineer:review run in parallel, then /engineer:sync-docs
#   → /engineer:coverage run sequentially, all against the branch

/engineer:pr
# → tests run, commit, PR opened, issue #42 moved to "in review"
```

Artifacts for an in-progress unit of work live in `.claude/work/<type>/<slug>/` (`context.md`, `architecture.md`, `plan.md`) — not just features: `<type>` follows [Conventional Commits](https://www.conventionalcommits.org/) (`feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `perf`, `build`, `ci`, `style`, `revert`), so a bug fix lives at `.claude/work/fix/<slug>/` and a docs update at `.claude/work/docs/<slug>/`, following the same `type/slug` shape as the branch name.

Doc-generation utilities live under `.claude/commands/bootstrap/` (`tech-docs`, `business-docs`, `index`) for bootstrapping the multi-file master-docs architecture in a new project from scratch.

## Agents (`.claude/agents/`)

| Agent | Used by | Role |
|---|---|---|
| `branch-master-docs-checker` | `/engineer:validate` (via `/engineer:pre-pr`) | checks the current branch against master docs |
| `branch-code-reviewer` | `/engineer:review` (via `/engineer:pre-pr`) | pre-PR code review |
| `branch-documentation-writer` | `/engineer:sync-docs` (via `/engineer:pre-pr`) | keeps user-facing docs (README, API reference, usage examples) in sync with code changes |
| `branch-test-planner` | `/engineer:coverage` (via `/engineer:pre-pr`) | finds missing test coverage on the branch |
| `adr-compliance-checker` | `/engineer:discover`, `/engineer:work` | validates code against the project's ADRs |
| `github-project-sync` | `/product:sync-github` | syncs `docs/business-context/features/` with GitHub Issues |
| `python-developer` | on demand | idiomatic Python implementation; checks real dependency versions and uses Context7 (if configured) before implementing against a library |
| `typescript-developer` | on demand | idiomatic TypeScript/JavaScript implementation (frontend or Node backend); same version-check + Context7 habit |

Add project- or stack-specific agents with `/meta:create-agent` rather than editing this set — keep the framework's own agent roster generic. `.claude/agents/` has no subdirectories (Claude Code doesn't discover agents in them), so `/meta:create-agent` names every agent it creates `project-*.md` by default — that prefix is what separates "this project's own" from the 8 unprefixed names above.

## Adopting CoDriDe in a project

1. Copy `.claude/` (and this file, merged with any project-specific instructions) into the target repo.
2. Run `/engineer:doctor` — confirms `gh` auth, the real default branch, whether a test suite exists, and what's already there on an existing project, before anything else runs on assumptions.
3. Run `/engineer:discover` once to generate `docs/technical-context/`.
4. Write master docs under `docs/business-context/` (use `/bootstrap:business-docs` and `/bootstrap:tech-docs` to bootstrap them).
5. From there, follow the pipeline above.

See [README.md](README.md) for the full walkthrough.
