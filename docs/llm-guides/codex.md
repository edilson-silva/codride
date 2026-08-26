# CoDriDe on Codex CLI

This guide is for adopting CoDriDe in a project using [Codex CLI](https://developers.openai.com/codex) instead of (or alongside) Claude Code or Gemini CLI. It assumes no familiarity with Claude Code — everything here is native to Codex CLI.

## What you copy

From the [CoDriDe repository](https://github.com/edilson-silva/codride), copy two things into your project:

```bash
cp -r codride/.agents codride/AGENTS.md /path/to/your-project/
```

- **`.agents/skills/`** — CoDriDe's skills, one directory per skill (`.agents/skills/<name>/SKILL.md`), named `<namespace>-<name>` (`engineer-context`, `product-spec`, `bootstrap-tech-docs`, etc.) to stay collision-free without a directory-based namespace. Two skills — `python-developer`, `typescript-developer` — keep a flat name since they were never namespaced.
- **`AGENTS.md`** — the pipeline overview, agent-to-skill mapping, and adoption steps, auto-loaded by Codex CLI (scanned from your working directory up to the repository root).

You never need Claude Code, `.claude/`, or `CLAUDE.md` for any of this — `.agents/` and `AGENTS.md` are complete and self-contained.

## Getting started

Same six steps as the Claude Code Quick Start, invoked as Codex Skills:

```
engineer-doctor      # pre-flight check
meta-preferences     # documentation language, extensible for more later
engineer-discover     # generates docs/technical-context/
                      # then write docs/business-context/ via bootstrap-business-docs
warm-up               # start a session
```

Invoke a skill by mentioning it explicitly (`$engineer-doctor`) or just describe your task in natural language — Codex will select the matching skill implicitly when your request matches its description. From there, the full pipeline works exactly as documented in the main [README](../../README.md) — same overall flow, same commands by name, just accessed differently. Work items land at `.codex/work/<type>/<slug>/` — a Codex-specific location, not shared with `.claude/work/` or `.gemini/work/` even on the same project.

## What's different from Claude Code

This target has two structural differences, not just cosmetic ones — read both before assuming a command behaves the way it does on Claude Code.

**No sub-agent delegation at all.** Claude Code has 8 core sub-agents, each a separate file the orchestrator can delegate to — including running some concurrently (`/engineer:pre-pr`'s `validate`+`review` step). Codex CLI has no delegation mechanism whatsoever, not even sequential delegation to a separate file. Six of those agents are each inlined directly into the one skill that used to invoke them — `engineer-validate`, `engineer-review`, `engineer-sync-docs`, `engineer-coverage`, `engineer-discover`, `engineer-work`, and `product-sync-github` (seven skills, since `engineer-discover` and `engineer-work` both independently carry a full copy of the sixth agent's ADR-compliance-checking logic, duplicated rather than shared — Codex has no reference mechanism for that either). The two on-demand agents with no fixed caller (`python-developer`, `typescript-developer`) became their own standalone skills instead, invoked directly rather than delegated to.

**No formal argument syntax.** Claude Code commands read a substituted `#$ARGUMENTS` block; Gemini CLI uses `{{args}}`. Codex Skills have neither — confirmed directly against Codex's own documentation, not assumed. A Skill has no templated way to receive parameters at all; it works entirely from the natural-language context of your request. Every skill that used to read an argument (a work-item slug, an issue number, a feature description) now instructs Codex to extract that information from what you actually typed, and to ask you rather than guess if it's ambiguous. In practice: instead of typing something equivalent to `/engineer:context feat/csv-order-export`, you'd say something like "start context for feat/csv-order-export" and the skill reads the slug from your sentence.

**Skill naming has no namespace.** `.agents/skills/<name>/` is flat — there's no subdirectory-per-namespace the way Claude Code (`.claude/commands/engineer/`) or Gemini CLI (`.gemini/commands/engineer/`) have. CoDriDe's skills are named `<namespace>-<name>` instead (`engineer-validate`, `product-validate` — both exist and don't collide) to preserve the same distinction without a folder to do it.

**No Artifacts support.** Same as Gemini CLI — Claude Code's `/meta:preferences` configures Artifacts (claude.ai-hosted pages) publishing; that feature doesn't exist here, so it's absent from `meta-preferences` entirely.

**No generated-target freshness check.** Same as Gemini CLI — the internal bookkeeping check that flags a stale generated target on Claude Code's `/engineer:doctor` doesn't apply to your project; it's excluded from this generation. If your copy of `.agents/`/`AGENTS.md` looks out of date, ask whoever maintains your CoDriDe source to regenerate it.

**`meta-create-agent` creates Codex-native skills.** It writes `.agents/skills/project-<name>/SKILL.md` files, with one deliberate omission: Claude Code sub-agents carry a `tools:` allowlist scoping exactly what that agent can access; Codex Skills have no equivalent tool-restriction field, so `meta-create-agent` doesn't try to invent one — a skill you create is instructions plus frontmatter, nothing more.

## Source of truth

Everything under `.agents/` and `AGENTS.md` is generated from CoDriDe's canonical Claude Code source, maintained upstream. Don't hand-edit generated files — changes are lost the next time the source is regenerated (a full overwrite, never a merge). If you need something changed, that happens upstream, in the CoDriDe repository itself.
