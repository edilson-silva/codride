# CoDriDe on GitHub Copilot CLI

This guide is for adopting CoDriDe in a project using [GitHub Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli) instead of (or alongside) Claude Code, Gemini CLI, or Codex CLI. It assumes no familiarity with Claude Code — everything here is native to Copilot CLI.

## What you copy

From the [CoDriDe repository](https://github.com/edilson-silva/codride), copy two things into your project:

```bash
mkdir -p /path/to/your-project/.github
cp -r codride/.github/agents /path/to/your-project/.github/
cp codride/.github/copilot-instructions.md /path/to/your-project/.github/copilot-instructions.md
```

Check that last line before running it: many repos already have a `.github/copilot-instructions.md` for GitHub's own Copilot code review — this would silently overwrite it. Merge the two by hand instead of blindly copying if your target repo already has one.

- **`.github/agents/`** — CoDriDe's commands *and* its 8 core agents, all in one flat directory, one `.agent.md` file each. Commands are named `<namespace>-<name>` (`engineer-context`, `product-spec`, `bootstrap-tech-docs`, etc.) to stay collision-free without a directory-based namespace; the 8 core agents keep their original flat names (`branch-code-reviewer`, `adr-compliance-checker`, etc.), since they were never namespaced commands to begin with.
- **`.github/copilot-instructions.md`** — the pipeline overview, agent roster, and adoption steps, auto-loaded by Copilot CLI at the start of every session in this project.

You never need Claude Code, `.claude/`, or `CLAUDE.md` for any of this — `.github/agents/` and `.github/copilot-instructions.md` are complete and self-contained.

## Getting started

Same six steps as the Claude Code Quick Start, invoked as Copilot agents:

```
engineer-doctor      # pre-flight check
meta-preferences     # documentation language, extensible for more later
engineer-discover     # generates docs/technical-context/
                      # then write docs/business-context/ via bootstrap-business-docs
warm-up               # start a session
```

Invoke an agent with `/agent-name` interactively, `--agent <name> --prompt "..."` programmatically, or just describe your task in natural language — Copilot will infer the matching agent from its `description`. From there, the full pipeline (product track, engineering track, `engineer-pre-pr`, `engineer-pr`) works exactly as documented in the main [README](../../README.md) — same overall flow, same commands by name, just accessed differently (see "No formal argument syntax" below). Work items land at `.copilot/work/<type>/<slug>/` — a Copilot-specific location, not shared with `.claude/work/`, `.gemini/work/`, or `.codex/work/` even on the same project.

## What's different from Claude Code

**One unified agent format, not two.** Claude Code splits commands (`.claude/commands/`) from sub-agents (`.claude/agents/`) into separate directories and file shapes. Copilot CLI has just one: `.agent.md`, distinguished by a `user-invocable` frontmatter flag — `true` for something you invoke directly (a command), `false` for something only another agent delegates to (a core agent). All 35 files — 27 commands, 8 core agents — live flat in `.github/agents/`. Two of the 8 core agents are the exception: `python-developer` and `typescript-developer` have no fixed calling command on Claude Code (they're used "on demand," invoked directly by a human), so both keep `user-invocable: true` here — the same carve-out Codex CLI makes by turning them into standalone Skills instead.

**Real subagent delegation — nothing is inlined.** Unlike Codex CLI, Copilot CLI does support an agent delegating to another agent, so this translation preserves every command→agent relationship exactly as it works on Claude Code: `engineer-validate` still delegates to `branch-master-docs-checker`, `product-sync-github` still delegates to `github-project-sync`, and so on — as separate files, not inlined logic.

**Orchestrator commands still run sequentially, not in parallel — but for a different reason than Gemini's.** `engineer-pre-pr` normally runs `engineer-validate` and `engineer-review` concurrently on Claude Code. Copilot CLI's subagent concurrency is real and configurable (not experimental the way Gemini's is) — but it's gated behind specific plan tiers and explicit settings (v1.0.66+, usage-based billing), so it can't be assumed present for every adopter. This generation defaults to sequential for that reason. If your plan has concurrency configured, you can enable parallel dispatch for `engineer-validate`/`engineer-review` yourself.

**No formal argument syntax.** Claude Code commands read a substituted `#$ARGUMENTS` block; Gemini CLI uses `{{args}}`. Copilot CLI agents have neither — confirmed directly against GitHub's own documentation, not assumed. An agent has no templated way to receive parameters at all; it works entirely from the natural-language context of your request or the `--prompt` flag. Every agent that used to read an argument (a work-item slug, an issue number, a feature description) now instructs Copilot to extract that information from what you actually typed, and to ask you rather than guess if it's ambiguous.

**Agent tool names are unverified.** Copilot CLI's own tool-name vocabulary (`read`, `search`, etc.) was sourced from a community example, not GitHub's own complete reference table, when this was generated. Each agent's `tools:` frontmatter field carries over Claude Code's tool names as a flagged placeholder, not a guess dressed up as fact — a comment above `tools:` says so. If an agent doesn't behave as expected, this is the first place to check.

**No Artifacts support.** Same as Gemini CLI and Codex CLI — Claude Code's `/meta:preferences` configures Artifacts (claude.ai-hosted pages) publishing; that feature doesn't exist here, so it's absent from `meta-preferences` entirely.

**No generated-target freshness check.** Same as Gemini CLI and Codex CLI — the internal bookkeeping check that flags a stale generated target on Claude Code's `/engineer:doctor` doesn't apply to your project; it's excluded from this generation. If your copy of `.github/agents/`/`.github/copilot-instructions.md` looks out of date, ask whoever maintains your CoDriDe source to regenerate it.

**`meta-create-agent` creates Copilot-native agents.** It writes `.github/agents/project-<name>.agent.md` files, using Copilot's real frontmatter — including a genuine `tools:` field (unlike Codex, which has none) and, for MCP-backed capabilities, `mcp-servers:` instead of an individual tool entry.

**Bonus: Copilot CLI also reads `AGENTS.md`, `CLAUDE.md`, and `GEMINI.md` directly, if present.** It combines and deduplicates them with `.github/copilot-instructions.md`, with no defined precedence order between them. This is a nice-to-have if your project happens to carry more than one CoDriDe target side by side — it isn't something this generation depends on, since a Copilot-only adopter never has those other files.

## Source of truth

Everything under `.github/agents/` and `.github/copilot-instructions.md` is generated from CoDriDe's canonical Claude Code source, maintained upstream. Don't hand-edit generated files — changes are lost the next time the source is regenerated (a full overwrite, never a merge). If you need something changed, that happens upstream, in the CoDriDe repository itself.
