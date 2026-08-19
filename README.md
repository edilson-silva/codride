# Codride

**Context Driven Development for Claude Code.**

Codride is a set of Claude Code commands and agents (living under `.claude/`) that structure how a project goes from a raw idea to a merged PR — with a persistent, versioned source of truth (**master docs**) that every feature is checked against, and GitHub Issues as the system of record for project management.

This repo *is* the framework: there's no application code here. You copy `.claude/` (and `CLAUDE.md`) into the project you actually want to build, and Codride's commands become available there.

---

## Why

Most AI-assisted workflows lose context between sessions: every feature re-derives "how do we build things here" from scratch, and there's nothing to check a change against except a reviewer's memory. Codride fixes this with two ideas:

1. **Master docs are the project's DNA.** A small set of living documents (business context + technical context) captures the decisions that matter — product strategy, personas, ADRs, conventions. Every feature gets checked against them before it's built (`/product:validate`) and again before it ships (`/engineer:validate`, part of `/engineer:pre-pr`).
2. **Every unit of work leaves a paper trail.** `/engineer:context` and `/engineer:architecture` write `context.md` and `architecture.md`, and `/engineer:plan` writes a phased `plan.md`, into `.claude/work/<type>/<slug>/` — and this isn't just for features. `<type>` follows [Conventional Commits](https://www.conventionalcommits.org/) (`feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `perf`, `build`, `ci`, `style`, `revert`), so a bug fix's work item lives at `.claude/work/fix/<slug>/`, a docs pass at `.claude/work/docs/<slug>/`, and so on — the same `type/slug` shape you'd use for the branch name. If work gets interrupted — a new chat, a different day, a different engineer — the next person reads its files and knows exactly where things stand.

Project management runs through **GitHub Issues** via the `gh` CLI — no separate PM tool to keep in sync by hand.

---

## Installing Codride in a project

1. Copy `.claude/` and `CLAUDE.md` into the target repo (merge `CLAUDE.md` with any project-specific instructions you already have there).
2. Make sure the [`gh` CLI](https://cli.github.com/) is installed and authenticated (`gh auth status`) — the whole product/engineering pipeline reads and writes GitHub Issues through it.
3. Run `/engineer:discover` once. It scans the codebase (and any existing ADRs) and writes `docs/technical-context/project-briefing.md` + `docs/technical-context/briefing/*.md`. This step is optional but every other command gets sharper once it's there.
4. Seed your master docs under `docs/business-context/` and `docs/technical-context/`. `/bootstrap:tech-docs` generates the technical side by analyzing your codebase. `/bootstrap:business-docs` generates the business side either by analyzing material you point it at, or — for a brand-new project with nothing to analyze yet — by running a founder/PM interview instead.
5. Start using the pipeline below. `/warm-up` is a good first command in any new session.

You don't need every piece on day one — `/engineer:context` and `/engineer:architecture` work fine without master docs or a discovery briefing; they just have less context to draw on.

---

## The pipeline

```
/warm-up
   │
   ├─ (once, optional) /engineer:discover
   │
   ├─ Product track:     /product:collect → refine → validate → spec
   │                      /product:brainstorm   (for open-ended decisions)
   │                      /product:sync-github  (keep docs ↔ GitHub Issues in sync)
   │                      /product:quick-spec   (spec an issue directly, no interview)
   │
   └─ Engineering track: /engineer:context → architecture → plan → work (× N phases) → pre-pr → pr
                          /engineer:bump  (release)

      /engineer:pre-pr is an orchestrator, not a check of its own — it just runs these
      4 standalone commands, each usable on its own too:
        /engineer:validate  ─┐ (branch vs. master docs)   parallel, read-only
        /engineer:review    ─┘ (code review)
        /engineer:sync-docs ─  (update project docs)      sequential, writes
        /engineer:coverage  ─  (missing test coverage)
```

A feature typically flows **left to right** through one track and then the other: an idea gets collected and refined on the product side, then handed to the engineering side once there's a clear spec to build against. Small, well-understood changes can skip straight to `/engineer:context`.

`/engineer:context` and `/engineer:architecture` are the two steps that used to be one `/engineer:start` command — split so each one's output matches its name, and so work interrupted right after `context.md` is approved can resume with a single command instead of re-running the whole interview. They're also not gated on a full product-track spec: for a technical decision that needs real architecture thinking before you're ready to commit to a formal PRD, run them early.

### Full walkthrough

```
/product:collect "customers want to export their order history as CSV"
# → creates GitHub issue #42

/product:refine 42
# → issue #42 rewritten as WHY / WHAT / HOW

/product:validate "CSV export of order history, issue #42"
# → confirms this doesn't violate any master doc; flags anything that does

/product:spec 42
# → issue #42 expanded into a full PRD (personas, NFRs, acceptance criteria, ...)

/engineer:context feat/csv-order-export
# → interview, then .claude/work/feat/csv-order-export/context.md

/engineer:architecture feat/csv-order-export
# → .claude/work/feat/csv-order-export/architecture.md, cross-checked against context.md

/engineer:plan feat/csv-order-export
# → .claude/work/feat/csv-order-export/plan.md, split into ~2h phases

/engineer:work .claude/work/feat/csv-order-export
# → implements Phase 1; issue #42 flips to "in progress"; repeat per phase

/engineer:pre-pr
# → /engineer:validate + /engineer:review run in parallel, then /engineer:sync-docs
#   → /engineer:coverage run sequentially; fix what they flag

/engineer:pr
# → tests run, commit, PR opened, issue #42 moved to "in review"
```

---

## Command reference

Commands are invoked as `/<folder>:<file>`, e.g. `.claude/commands/product/spec.md` → `/product:spec`. `/warm-up` lives at the top level, so it's just `/warm-up`.

### Setup

#### `/warm-up`
Loads both halves of the master docs — product (`docs/business-context/`) and engineering (`docs/technical-context/`) — plus the root `README.md`, so the session starts with the right context loaded. It reads index/entry-point files only, not everything they point to: `docs/business-context/index.md`, then either `docs/technical-context/project-briefing.md` (if `/engineer:discover` was run) or `docs/technical-context/index.md` (if `/bootstrap:tech-docs` was run), whichever exists. If a piece doesn't exist yet, it says so and moves on — it never fails just because the project hasn't bootstrapped everything.

- **Usage**: `/warm-up` (project name is optional, only useful in a multi-project workspace)
- **Tips**: run this at the start of any session where you'll touch product or architecture decisions. Skip it for a one-line bug fix. It intentionally doesn't front-load every file under `docs/` — just the indexes, so later commands can pull in specific files as needed without wasting context up front.

#### `/engineer:discover`
Scans the codebase once (or incrementally) and writes `docs/technical-context/project-briefing.md` plus `docs/technical-context/briefing/{critical-rules,adrs-summary,backend-conventions,tech-stack}.md`. Detects ADRs, infers architectural conventions (enum/type locations, repository vs. service patterns, naming), and identifies the stack from the manifest file.

- **Usage**: `/engineer:discover`, or `/engineer:discover --verbose` for a detailed run
- **When to run it**: once early in a project, then again whenever ADRs or the backend architecture change meaningfully. It's non-destructive — re-running it updates incrementally rather than overwriting.
- **Tips**: write your ADRs *before* running this if you can — the more decisions are documented, the more useful `critical-rules.md` becomes, and the sharper `adr-compliance-checker` gets during `/engineer:work`.

### Bootstrapping master docs (`bootstrap`)

These three generate the multi-file master-docs architecture from scratch by analyzing your codebase and any material you point them at. Use them once per project, then maintain the output by hand (or via `/engineer:sync-docs`) as things change.

#### `/bootstrap:tech-docs`
Generates the full technical-context architecture (project charter, ADRs, AI dev guide, codebase navigation, business logic, API spec, contributing guide, troubleshooting) under `docs/technical-context/`.

- **Usage**: `/bootstrap:tech-docs <links to repos/files to analyze>`
- **Tips**: point it at the actual codebase, not just a description — it reads build configs, ORM/DB choices, test frameworks, and CI setup directly.

#### `/bootstrap:business-docs`
Generates the full business-context architecture (personas, journey, voice of customer, product strategy, feature catalog, competitive landscape, sales/messaging guidance) under `docs/business-context/`. Works two ways: **analysis mode** mines material you point it at; **collection mode** runs a founder/PM interview instead, for a brand-new project with nothing to analyze yet.

- **Usage**: `/bootstrap:business-docs <links to product docs, support tickets, existing PRDs, etc.>` for analysis mode, or `/bootstrap:business-docs` with no arguments for collection mode.
- **Tips**: feed it real material whenever you have it (support tickets, sales call notes, competitor pages) — the richer the input, the more useful `CUSTOMER_PERSONAS.md` and `COMPETITIVE_LANDSCAPE.md` end up being for `/product:brainstorm` later. In collection mode, everything it generates is explicitly marked as an unvalidated hypothesis — re-run it (or use `/product:brainstorm`) once you have real customers to confirm or revise those assumptions, rather than leaving them marked as hypotheses forever.

#### `/bootstrap:index`
Builds or refreshes an `index.md` pointing to every useful documentation file. Detects whether this is a single project (the normal Codride shape) or a multi-project docs meta-repo, and adapts accordingly.

- **Usage**: `/bootstrap:index` (rebuild the index for this project, or the root index across `projects/` in meta-repo mode) or `/bootstrap:index <project-name>` (meta-repo mode: rebuild one project's index after restructuring)
- **Tips**: in single-project mode you can pass a subfolder to focus the refresh on, instead of rebuilding the whole thing.

### Product track

#### `/product:collect`
Captures a raw idea or bug report as a GitHub issue, with just enough clarity to recall it later — no full spec yet.

- **Usage**: `/product:collect "users can't reset their password if their email has a plus-alias"`
- **Tips**: use this the moment an idea comes up, even half-formed. It's cheap and keeps ideas from getting lost; refine later, only when you're ready to actually build it.

#### `/product:refine`
Turns a collected requirement into a structured WHY / WHAT / HOW document, through a clarifying-question dialogue.

- **Usage**: `/product:refine 42` (a GitHub issue number) or `/product:refine <path/to/file.md>`
- **Tips**: don't skip straight to `/product:spec` for anything non-trivial — refine first. The WHY section is what keeps `/product:spec` and `/engineer:context` from re-litigating "why are we building this" three commands later.

#### `/product:validate`
Validates one or more described features against the project's master docs, reporting what's aligned and what contradicts a specific master doc (with a citation).

- **Usage**: `/product:validate "add social login via Google and GitHub"`
- **Tips**: run this *before* `/product:spec`, not after — catching a master-doc conflict before writing a full PRD saves the rework. It never edits anything unless you ask it to.

#### `/product:spec`
Expands a validated requirement into a full PRD: product overview, functional requirements (numbered `FR-01`, `FR-02`, ...) with BDD acceptance criteria, non-functional requirements, UX considerations, technical considerations, risks, constraints. Also saves `docs/business-context/features/<slug>.md` in the format `/product:sync-github` expects — this is the step that actually populates that folder.

- **Usage**: `/product:spec 42`
- **Tips**: it deliberately skips sections that don't apply — don't fight it to fill in a template for a two-line change. It edits the source issue/file in place once you approve its understanding, and keeps the feature file in sync on every re-run.

#### `/product:brainstorm`
A structured, deliberately adversarial brainstorming session for open-ended product or business decisions — generates real alternatives, trade-off and risk matrices, and a grounded recommendation, then stops for human review.

- **Usage**: `/product:brainstorm "should we build a native mobile app or invest in the PWA?"`
- **Tips**: reserve this for decisions with real alternatives worth weighing — it's the heaviest command in the framework. It reads `PRODUCT_STRATEGY.md` / `CUSTOMER_PERSONAS.md` / `COMPETITIVE_LANDSCAPE.md` if they exist, so it's most useful once `/bootstrap:business-docs` has been run. For a purely technical "is this feasible" question rather than a business decision, use `/engineer:context` + `/engineer:architecture` instead.

#### `/product:quick-spec`
Creates a fully-specced GitHub issue directly from a task description, without the multi-step collect → refine → spec dialogue — useful for well-understood work. Also saves `docs/business-context/features/<slug>.md`, same as `/product:spec` does, unless the task is too small to be worth tracking as a feature.

- **Usage**: `/product:quick-spec "add rate limiting to the public API, 100 req/min per API key"`
- **Tips**: use this for work you already understand well; use `/product:collect` + `/product:refine` when you don't. It can split a complex task into a parent issue with child issues — it'll tell you before doing so.

#### `/product:sync-github`
Keeps `docs/business-context/features/*.md` in sync with this repo's GitHub Issues — creates missing issues, updates ones that drifted, flags orphans. Always previews the diff before writing anything.

- **Usage**: `/product:sync-github`, `/product:sync-github module=Billing`, or `/product:sync-github preview`
- **Tips**: this reads from `docs/business-context/features/`, which `/product:spec` and `/product:quick-spec` populate — run one of those first for any feature you want tracked. Run `preview` mode first the first time you use it on a repo, to see how it matches existing issues before it starts creating things. It never overwrites a manually-edited issue body outside the section it owns.

### Engineering track

#### `/engineer:context`
Kicks off a unit of work: an interview to build shared understanding, written to `context.md`. Not just for features — a fix or a chore gets the same treatment when it's substantial enough to warrant one. First of a two-step pair with `/engineer:architecture`.

- **Usage**: `/engineer:context feat/csv-order-export` (or `fix/password-reset-plus-alias`, `docs/api-reference`, ...) — argument is `<type>/<slug>`, matching the work item folder and branch name
- **Tips**: if `docs/technical-context/project-briefing.md` exists, this command asks which backend areas are impacted so it loads only the relevant ADRs — answer precisely, it's there to save tokens, not to slow you down. It stops after `context.md` and waits for your explicit go-ahead before you run `/engineer:architecture` — it will not design the architecture on its own.

#### `/engineer:architecture`
Reads `context.md` and designs the implementation, written to `architecture.md`, with a mandatory consistency check between the two documents before you approve. Fails fast and tells you to run `/engineer:context` first if that file doesn't exist yet.

- **Usage**: `/engineer:architecture feat/csv-order-export`
- **Tips**: the consistency check exists because `context.md` and `architecture.md` drifting apart (different file lists, different numbers, one says "modify X" and the other says "delete X") is a real, recurring failure mode — don't skip past its report. It stops and waits for `/engineer:plan` to be run explicitly.

#### `/engineer:plan`
Turns `context.md` + `architecture.md` into a phased `plan.md`, each phase sized to roughly 2 hours of human work, resumable if interrupted.

- **Usage**: `/engineer:plan feat/csv-order-export`
- **Tips**: if research during planning surfaces a new architectural decision, it'll update `architecture.md` too and flag the change — don't be surprised if both files move. Like `/engineer:architecture`, it stops and waits for `/engineer:work` to be run explicitly.

#### `/engineer:work`
Executes the next phase of `plan.md`, keeps the GitHub issue's status label in sync in real time, and — if `context.md` has critical ADR rules loaded — implements proactively against them instead of fixing violations after the fact.

- **Usage**: `/engineer:work .claude/work/feat/csv-order-export`
- **Tips**: it pauses after each phase for you to validate before starting the next — don't skip that checkpoint even when it feels obvious, it's what keeps `plan.md` an accurate record if the work gets picked up later by someone else (or by you, in three weeks).

#### `/engineer:pre-pr`
An orchestrator, not a check of its own: runs `/engineer:validate` and `/engineer:review` in parallel (both read-only, neither depends on the other), then `/engineer:sync-docs` and `/engineer:coverage` sequentially (both write to the branch, so they don't run concurrently with each other or with the first two) — and helps you act on their combined feedback.

- **Usage**: `/engineer:pre-pr`
- **Tips**: run this before `/engineer:pr`, not as a substitute for `/engineer:work`'s own per-phase checkpoints — it's a final sweep across the whole branch, not a phase-by-phase review. If you only need one of the four checks, run it directly instead (see below) — `/engineer:pre-pr` is the recommended default, not a requirement.

#### `/engineer:validate`
A one-line shortcut that invokes `branch-master-docs-checker` directly, checking the current branch against the project's master docs.

- **Usage**: `/engineer:validate`
- **Tips**: not to be confused with `/product:validate`, which checks a *spec* against master docs before anything is built — this one checks the *branch*, after the fact.

#### `/engineer:review`
A one-line shortcut that invokes `branch-code-reviewer` directly, reviewing the branch's changes for quality, bugs, and best practices.

- **Usage**: `/engineer:review`
- **Tips**: good to run mid-feature on a partially-done branch, not just right before a PR — catching an issue earlier is cheaper than catching it in `/engineer:pre-pr`.

#### `/engineer:sync-docs`
A one-line shortcut that invokes `branch-documentation-writer` directly, updating project docs to match the branch's code changes.

- **Usage**: `/engineer:sync-docs`
- **Tips**: use this when you only need docs caught up — e.g. after a docs-only PR review comment — rather than a full pre-PR sweep.

#### `/engineer:coverage`
A one-line shortcut that invokes `branch-test-planner` directly, identifying missing test coverage for the branch's changes.

- **Usage**: `/engineer:coverage`
- **Tips**: run this right after finishing a phase in `/engineer:work`, while the code you just wrote is still fresh, instead of waiting for `/engineer:pre-pr` to surface gaps at the end.

#### `/engineer:pr`
Runs tests, commits, opens the PR, moves the GitHub issue to "in review", and — after a short wait — triages automated code-review bot comments with you.

- **Usage**: `/engineer:pr`
- **Tips**: it only stages files it actually changed (never `git add .`), and it won't mention AI/Claude tooling in the PR description. It stops for your explicit approval before applying any bot-suggested fix.

#### `/engineer:bump`
Bumps the project's semver version, detecting whether the project uses `pyproject.toml`, `package.json`, or both.

- **Usage**: `/engineer:bump`
- **Tips**: it decides major/minor/patch from your description of the change — say what actually changed ("removed the old `/v1/users` endpoint" vs. "added CSV export"), don't just say "bump the version."

#### `/engineer:adr`
Drafts a new Architecture Decision Record under `docs/technical-context/adr/`, checking first for conflicts with or supersession of existing ADRs. The only command in the framework that *writes* an ADR — everything else (`/engineer:discover`, `adr-compliance-checker`, `/engineer:architecture`) only reads them.

- **Usage**: `/engineer:adr "use event sourcing for the order aggregate"`
- **Tips**: use it for decisions that constrain future work (a technology choice, a pattern adopted or rejected), not implementation details. `/engineer:plan` and `/engineer:architecture` will prompt you to run this when research surfaces a decision significant enough to record. Run `/engineer:discover` afterward so the new ADR gets picked up by the technical briefing.

### Meta

#### `/meta:create-agent`
Creates a new sub-agent under `.claude/agents/`, walking through purpose, minimal tool selection, and system-prompt design.

> **Naming**: `.claude/agents/` is a flat namespace — Claude Code doesn't discover agents in subdirectories, so there's no folder to keep project agents out of the framework's own. The filename does that job instead: **every agent this command creates is named `project-<name>.md` by default** (e.g. `project-notion-specialist.md`), so it's obvious at a glance which agents are Codride's portable core (the 8 listed above, unprefixed) and which are specific to this project.

- **Usage**: `/meta:create-agent "an agent that audits our GraphQL schema for breaking changes before merge"` → creates `project-graphql-schema-auditor.md`
- **Tips**: this is how you extend Codride with project- or stack-specific agents (a NestJS specialist, a Terraform reviewer, whatever your project needs) without bloating the framework's own generic agent roster. Default to the minimum tool set — a checker that never writes files needs `Read, Glob, Grep, Bash`, nothing more.

---

## Agents

These 8 are Codride's portable core — unprefixed names. Anything created via `/meta:create-agent` is project-specific and gets a `project-` prefix instead (see below) — that prefix is the only thing distinguishing the two, since `.claude/agents/` doesn't support subdirectories.

| Agent | Invoked by | Tools | Role |
|---|---|---|---|
| `branch-master-docs-checker` | `/engineer:validate` (also via `/engineer:pre-pr`) | `Read, Glob, Grep, Bash` | Checks the branch against master docs |
| `branch-code-reviewer` | `/engineer:review` (also via `/engineer:pre-pr`) | `Read, Glob, Grep, Bash` | Pre-PR code review (quality, bugs, security, perf) |
| `branch-documentation-writer` | `/engineer:sync-docs` (also via `/engineer:pre-pr`) | `Read, Write, Edit, Glob, Grep, Bash` | Keeps docs in sync with code changes |
| `branch-test-planner` | `/engineer:coverage` (also via `/engineer:pre-pr`) | `Read, Glob, Grep, Bash, Write, Edit` | Finds missing test coverage on the branch |
| `adr-compliance-checker` | `/engineer:discover`, `/engineer:work` | `Read, Glob, Grep, Bash` | Validates code against the project's ADRs |
| `github-project-sync` | `/product:sync-github` | `Read, Bash, Glob, Grep` | Syncs feature docs with GitHub Issues |
| `python-developer` | on demand — invoked automatically for non-trivial Python work, or ask for it explicitly | `Read, Write, Edit, Bash, Glob, Grep, WebSearch, Context7*` | Idiomatic Python implementation |
| `typescript-developer` | on demand — invoked automatically for non-trivial TS/JS work, or ask for it explicitly | `Read, Write, Edit, Bash, Glob, Grep, WebSearch, Context7*` | Idiomatic TypeScript/JavaScript (frontend or Node) |

Each agent's tool list is deliberately minimal — checkers don't get `Write`, implementers do. Keep that convention when adding agents with `/meta:create-agent`.

`*` `python-developer` and `typescript-developer` list the Context7 MCP tools (`mcp__context7__resolve-library-id`, `mcp__context7__get-library-docs`) so they can use it when it's configured — see Optional MCPs below. Listing them doesn't require the MCP to be present; the agents check the real dependency version first (manifest/lockfile) and fall back to `WebSearch` if Context7 isn't set up.

---

## Optional MCPs

Codride's core loop needs nothing beyond `gh` — no MCP server is required. Two are worth adding for specific, real gains; neither is assumed by default, and commands that mention them always say "if available."

- **[Context7](https://github.com/upstash/context7)** — up-to-date library/framework documentation lookup, scoped to the actual version in use. Referenced by `/engineer:architecture`, `/engineer:plan`, and `/product:brainstorm` as "if available," and wired directly into `python-developer` and `typescript-developer`'s tool lists — those two always check the real dependency version first (manifest/lockfile, or `tech-stack.md` if `/engineer:discover` has run), then use Context7 for that version's docs if it's configured, or `WebSearch` if it isn't. Worth adding once you're tired of an agent guessing at an API that changed since its training data.
- **A Playwright MCP** (e.g. [`@playwright/mcp`](https://github.com/microsoft/playwright-mcp)) — browser automation. This is what closes the loop on the BDD acceptance criteria `/product:spec` writes (`Given/When/Then`): without it, those scenarios only ever get verified by unit/integration tests; with it, an agent can actually drive a browser through the scenario against a running app. Only relevant for projects with a web UI — skip it for a pure API or CLI project.

Don't add MCPs for services this framework deliberately doesn't integrate with — see the `gh`-only rationale in Best practices below.

---

## Directory conventions

```
docs/
├── business-context/           # master docs: strategy, personas, feature catalog
│   ├── index.md                 # entry point, generated by /bootstrap:business-docs
│   ├── features/                 # one .md per feature, populated by /product:spec or /product:quick-spec, kept in sync via /product:sync-github
│   └── brainstorm/               # /product:brainstorm session output
└── technical-context/          # shape depends on which command generated it:
    ├── project-briefing.md      #   /engineer:discover  → compact briefing (+ briefing/*.md below)
    ├── briefing/                 #   critical-rules, adrs-summary, backend-conventions, tech-stack
    ├── index.md                 #   /bootstrap:tech-docs → entry point for the fuller set below
    ├── project_charter.md
    ├── adr/                      # Architecture Decision Records
    └── CODEBASE_GUIDE.md, BUSINESS_LOGIC.md, API_SPECIFICATION.md, CONTRIBUTING.md, TROUBLESHOOTING.md
```

`docs/technical-context/` normally has just one of the two shapes above, not both — `/engineer:discover` is the lightweight, pipeline-integrated option; `/bootstrap:tech-docs` is the comprehensive, general-reference option. `/warm-up` checks for either.

```
.claude/
├── agents/                          # the 8 agents above
├── commands/                        # engineer/, product/, bootstrap/, meta/, warm-up.md
├── work/<type>/<slug>/               # context.md, architecture.md, plan.md per in-flight work item
│   ├── feat/csv-order-export/        # e.g. a feature
│   ├── fix/password-reset-plus-alias/ # e.g. a bug fix
│   └── docs/api-reference/            # e.g. a docs update
└── rules/product-agent.mdc          # always-on PM/architect persona
```

`<type>` follows [Conventional Commits](https://www.conventionalcommits.org/) (`feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `perf`, `build`, `ci`, `style`, `revert`) — work items aren't just for features. Use whichever type matches the work, and name the branch the same way (`type/slug`) so the work item folder, the branch, and the eventual commit type all line up.

---

## Best practices

- **Don't skip the pauses.** `/engineer:context`, `/engineer:architecture`, `/engineer:plan`, and `/product:brainstorm` all stop and wait for explicit approval at key points. That's by design — it's the checkpoint that keeps `context.md`/`architecture.md`/`plan.md` trustworthy for whoever (or whatever session) picks up the work next.
- **Run `/engineer:discover` before you need it, not during a feature.** It's meant to run ahead of time so `/engineer:context` has a briefing to load; running it mid-feature works too, but you lose the token savings from selective context loading.
- **Keep master docs as the tie-breaker, not a formality.** `/product:validate` and `branch-master-docs-checker` are only useful if contradicting them is treated as a real signal to stop and discuss, not noise to click through.
- **Small changes don't need the full pipeline.** A one-line fix doesn't need `/product:collect` → `/product:spec` → `/engineer:context` → `/engineer:architecture` → `/engineer:plan` — go straight to a branch and `/engineer:pre-pr` before opening the PR. Save the full pipeline for work substantial enough to benefit from a written plan.
- **A pre-spec architecture question doesn't need a separate command.** If you need real architecture thinking before committing to a full PRD, just run `/engineer:context` + `/engineer:architecture` early — they don't require a finished spec, only a clear enough idea of what's being explored.
- **Extend, don't edit, the agent roster.** Use `/meta:create-agent` for anything project- or stack-specific (a framework specialist, a domain-specific linter) — it prefixes the file `project-*` by default, so it's never ambiguous which agents are Codride's portable core versus this project's own. Keep the 8 shipped agents generic and unprefixed so this framework stays portable across projects.
- **`gh` is the integration point, not a GraphQL API key.** Every GitHub interaction in this framework goes through the `gh` CLI, authenticated as whoever is running Claude Code — there's no token to manage or rotate in `.claude/`.
