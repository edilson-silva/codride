---
description: Generate this project's command/agent set for another LLM CLI, translated from the canonical .claude/ source
argument-hint: <target> — gemini, codex, or copilot
---

# Generate Target

This command translates CoDriDe's canonical, hand-authored source — `.claude/commands/`,
`.claude/agents/`, root `CLAUDE.md`, `.claude/rules/*.md` — into the native format of one other
LLM CLI at a time. Run it here, in this repo, as CoDriDe's maintainer; it's never meant to be run
by someone adopting the framework. An adopter who wants Gemini support (for example) just copies
the generated `.gemini/` output out of this repo, the same way they copy `.claude/` today — they
never need Claude Code, or this command, to get there.

`.claude/` is never written by this command and never changes as a result of running it — it's
always the input, never the output.

<target>
#$ARGUMENTS
</target>

If the argument isn't exactly one of `gemini`, `codex`, or `copilot`, stop and ask which target
was meant rather than guessing.

## Algorithm

### 1. Read the canonical source in full

Every file under `.claude/commands/`, every file under `.claude/agents/`, root `CLAUDE.md`, every
file under `.claude/rules/`. Don't work from partial memory of these files — read them now, this
run, since they may have changed since the last generation.

### 2. Look up the target's mapping rules

Use the subsection below matching the requested target. Each was seeded from research and may
have been refined since by that target's own branch — trust what's written there over any general
assumption about how these tools work.

### 3. Translate each command

For every `.claude/commands/<namespace>/<name>.md` **except this file itself**
(`.claude/commands/meta/generate-target.md`), produce the target's equivalent — its own command
format, or its closest fold-in if the target has no separate command layer (Copilot: every
CoDriDe command becomes a Copilot `.agent.md`). This command is maintainer-side only and has no
meaningful equivalent to produce for a target — an adopter using that target has no canonical
`.claude/` source to run it against.

This includes rewriting every `.claude/`-rooted path a command's body references.
`.claude/work/<type>/<slug>/` — found in `engineer/context.md`, `architecture.md`, `plan.md`,
`work.md`, `doctor.md`, and anywhere else a work item is read or written — becomes
`<target-root>/work/<type>/<slug>/`, by the exact same rule as every other path. **No special
case for this one.** Work-item folders are per-tool, not a shared fixed location: a centralized,
untranslated path was considered and deliberately rejected, precisely because it would have
needed a carve-out like this to remember — treating it as just another path to rewrite is what
keeps the algorithm free of exceptions. Don't skip it.

### 4. Translate each subagent

For every `.claude/agents/<name>.md`:

- **If the target has a native subagent primitive**: translate directly, preserving the `tools:`
  allowlist using the target's own tool vocabulary. If a CoDriDe tool name has no target
  equivalent, don't guess a mapping — flag it in the summary (step 9) instead.
- **If the target has no subagent primitive at all** (Codex, as of this writing): fold every
  command that currently invokes subagents into a single sequential prompt. Inline each invoked
  agent's instructions, in the same order the orchestrator invokes them today, into that one
  target file. No separate "agent" files exist for that target.

### 5. Collapse parallelism to sequential wherever the target's parallel dispatch isn't confirmed reliable

This is the default for any target until its own branch confirms otherwise. An orchestrator like
`/engineer:pre-pr`, which today runs two subagents concurrently in its Step 1, generates as a
straight numbered sequence for that target instead: check 1, then check 2, then check 3, then
check 4 — same content, no concurrency claim.

### 6. Fold auto-loaded instructions

Concatenate `CLAUDE.md` plus every `.claude/rules/*.md` file, in a fixed order, into the target's
single auto-loaded instruction file — with a clear heading per source file so provenance isn't
lost. If the target supports referencing other files directly instead of physical concatenation
(e.g. Copilot's `@relative/path` includes), prefer that over duplicating content.

### 7. Write the output as a full, clean overwrite of only what this command previously generated — never an incremental merge, and never a delete of anything it didn't create itself

Before writing, check `.claude/.generation-log.md` for that target's most recent file manifest
(step 8 records exactly which files were written, not just a hash — this is what makes a safe
overwrite possible). Delete every file listed in that manifest, then write the fresh translation
from scratch. If this is the first generation for that target, there's nothing to delete yet —
just write.

Do the delete-then-write on every run, first time or re-run alike. The canonical source keeps
changing — commands get removed, agents get renamed, tool lists change — and a partial or
diffing write would leave orphaned target files behind (a stale
`.gemini/commands/product/spec.toml` for a command that no longer exists in `.claude/`, for
example). Full overwrite is also the only approach consistent with this command being LLM-driven
rather than deterministic: there's no reliable diff between two non-byte-reproducible runs to
compute, so don't attempt one.

**If the target's auto-loaded instruction file already exists on disk (e.g. `AGENTS.md` for
Codex, `.github/copilot-instructions.md` for Copilot) but isn't listed in a prior manifest for
this target, stop and ask for explicit confirmation before overwriting or deleting it.** It
likely predates this command — hand-authored by whoever owns this repo for an unrelated reason —
and this command has no business silently destroying content it didn't create. This is the one
case where "only touch what I generated" and "full overwrite" could conflict; resolve it by
asking, never by guessing.

Write to the target's fixed, conventional path(s) at repo root. Never nest the output under a
CoDriDe-specific folder — each target tool only looks in its own fixed location, and the
generated tree needs to sit exactly where that tool expects it.

### 8. Append a line to `.claude/.generation-log.md`

Record: target name, timestamp, `git rev-parse HEAD` at generation time, and the full list of
file paths written this run — the manifest step 7 depends on to know what it's safe to delete on
the next run. This is also what lets `/engineer:doctor` later detect whether a target has
drifted from the current canonical source.

### 9. Report a summary

State what was generated and where. Then — explicitly, not silently — list anything that had no
clean mapping: unmatched tool names, ambiguous instructions, or target-format details this
command's own subsection didn't cover with confidence. A human should be able to act on this list
without having to re-derive it by reading the generated output themselves.

## Target-specific rules

### Gemini CLI

- **Commands** → TOML files under `.gemini/commands/<namespace>/<name>.toml` (project-level),
  namespaced by subdirectory exactly like `.claude/commands/` (`git/commit.toml` → `/git:commit`).
  Fields: `prompt`, `description`. **Unconfirmed — verify before first real run**: the exact
  argument-placeholder syntax for the `prompt` field (CoDriDe's `#$ARGUMENTS` substitution needs a
  Gemini-native equivalent).
- **Subagents** → `.md` + YAML frontmatter under `.gemini/agents/` (project-level, committable),
  with real `tools:`/MCP scoping.
- **Auto-loaded instructions** → root `GEMINI.md`.
- **Parallelism**: native subagent parallel dispatch exists but is experimental with known issues
  as of this writing — apply the sequential-by-default rule (step 5) unless the Gemini-specific
  branch confirms it's safe to rely on for a given orchestrator.

### Codex CLI

- **Commands** → Skills (`SKILL.md`-style, semantic/implicit invocation — not a `/namespace:name`
  slash syntax). The old `~/.codex/prompts/*.md` mechanism is deprecated; don't generate for it.
  **Unconfirmed — verify before first real run**: the exact project-level Skills directory (only
  the deprecated global `~/.codex/prompts/` path is confirmed).
- **Subagents** → none. Always apply step 4's collapse rule — every orchestrator becomes one
  sequential Skill file with all invoked agents' instructions inlined in order.
- **Auto-loaded instructions** → a single `AGENTS.md` per directory level (first non-empty file
  wins as you walk up the tree) — no directory-of-files pattern, so step 6's concatenation is
  mandatory here, not optional.
- **Parallelism**: not applicable — there's no subagent primitive to run concurrently in the first
  place.

### GitHub Copilot CLI

- **Commands and subagents** both fold into `.agent.md` files (`name`, `description`, `tools`,
  `model`, `mcp-servers` fields) — Copilot has no separate command layer, so every CoDriDe command
  becomes a Copilot agent, invoked via `/agent`, natural language, or auto-inference.
  **Unconfirmed — verify before first real run**: the conventional directory `.agent.md` files
  live in at the project level.
- **Auto-loaded instructions** → `.github/copilot-instructions.md`. Copilot can also `@`-include
  other files directly (including `AGENTS.md`/`CLAUDE.md`) — the Copilot-specific branch should
  evaluate using includes instead of full concatenation (step 6's fallback path) before assuming
  physical concatenation is necessary here.
- **Parallelism**: `.agent.md` files have real tool scoping, but parallel dispatch across them is
  unconfirmed either way — apply the sequential-by-default rule (step 5) until the Copilot-specific
  branch confirms otherwise.
