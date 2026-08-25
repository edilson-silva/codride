# CoDriDe on Gemini CLI

This guide is for adopting CoDriDe in a project using [Gemini CLI](https://github.com/google-gemini/gemini-cli) instead of (or alongside) Claude Code. It assumes no familiarity with Claude Code — everything here is native to Gemini CLI.

## What you copy

From the [CoDriDe repository](https://github.com/edilson-silva/codride), copy two things into your project:

```bash
cp -r codride/.gemini codride/GEMINI.md /path/to/your-project/
```

- **`.gemini/commands/`** — CoDriDe's slash commands (`/engineer:context`, `/product:spec`, etc.), one TOML file per command, namespaced the same way as Claude Code's (`engineer/`, `product/`, `bootstrap/`, `meta/`).
- **`.gemini/agents/`** — the 8 core CoDriDe agents (code reviewer, master-docs checker, documentation writer, test planner, ADR checker, GitHub sync, Python/TypeScript developers).
- **`GEMINI.md`** — the pipeline overview, agent roster, and adoption steps, auto-loaded by Gemini CLI at the start of every session in this project.

You never need Claude Code, `.claude/`, or `CLAUDE.md` for any of this — `.gemini/` and `GEMINI.md` are complete and self-contained.

## Getting started

Same six steps as the Claude Code Quick Start, just run through Gemini CLI:

```
/engineer:doctor        # pre-flight check
/meta:preferences       # documentation language, extensible for more later
/engineer:discover      # generates docs/technical-context/
                         # then write docs/business-context/ via /bootstrap:business-docs
/warm-up                # start a session
```

From there, the full pipeline (product track, engineering track, `/engineer:pre-pr`, `/engineer:pr`) works exactly as documented in the main [README](../../README.md) — command names, arguments, and the overall flow are identical. Work items land at `.gemini/work/<type>/<slug>/` (mirroring `.claude/work/` on Claude Code, but not shared with it — see "What's different" below).

## What's different from Claude Code

**Orchestrator commands run sequentially, not in parallel.** `/engineer:pre-pr` normally runs `/engineer:validate` and `/engineer:review` concurrently on Claude Code. On Gemini CLI, this generation runs every step of `/engineer:pre-pr` one after another instead. This isn't a CoDriDe limitation — Gemini CLI does have native subagent parallel dispatch, but it's experimental with known issues as of when this was generated, so CoDriDe defaults to the safer sequential path until that matures. Expect `/engineer:pre-pr` to take somewhat longer here than the same sweep on Claude Code; nothing about the checks themselves is different.

**Agent tool names are unverified.** Gemini CLI's own tool-name vocabulary wasn't confirmed when this was generated — each of the 8 agents' `tools:` frontmatter field carries over Claude Code's tool names (`Read`, `Glob`, `Grep`, `Bash`, etc.) as a flagged placeholder, not a guess dressed up as fact. Each agent file has a comment above `tools:` saying so. If an agent doesn't behave as expected, this is the first place to check — confirm the real tool names against your Gemini CLI version and update accordingly.

**No Artifacts support.** Claude Code's `/meta:preferences` includes a setting for publishing Artifacts (claude.ai-hosted pages) — that feature doesn't exist in Gemini CLI, so it's absent here entirely. `/meta:preferences` on this generation only configures documentation language.

**No generated-target freshness check.** Claude Code's `/engineer:doctor` includes a check for whether a generated target (like this one) has drifted from the upstream canonical source. That check is internal bookkeeping for whoever maintains CoDriDe itself and doesn't apply to your project — it's excluded from this generation. If you suspect your copy of `.gemini/`/`GEMINI.md` is out of date, ask whoever maintains your CoDriDe source to regenerate and re-copy it; that's not something to do from inside your own project.

**`/meta:create-agent` creates Gemini-native agents.** It writes `.gemini/agents/project-<name>.md` files with the same unverified-tools caveat as the 8 core agents — genuinely adapted for Gemini CLI, not a mechanical translation of the Claude Code version.

## Source of truth

Everything under `.gemini/` and `GEMINI.md` is generated from CoDriDe's canonical Claude Code source, maintained upstream. Don't hand-edit generated files — changes are lost the next time the source is regenerated (a full overwrite, never a merge). If you need something changed, that happens upstream, in the CoDriDe repository itself.
