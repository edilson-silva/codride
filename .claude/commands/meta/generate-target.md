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
  allowlist using the target's own tool vocabulary. If the target's real tool-name vocabulary
  hasn't been confirmed (e.g. via research, the way `{{args}}` was confirmed for Gemini), don't
  invent target-native names — carry the CoDriDe/Claude Code tool names over verbatim instead,
  with an explicit inline comment directly above the `tools:` field stating they're unverified
  and need confirming against the target's real vocabulary. This is more useful to whoever reads
  the generated file than an empty or omitted `tools:` field would be. Flag it in the run's
  summary (step 9) as well — the inline comment and the summary note are both required, not one
  or the other.
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

### 9. Cross-file consistency check, then report a summary

Before reporting, grep the freshly generated output for every concept this run decided to drop,
adapt, or reword for this target (e.g. a feature with no target equivalent, a reworded opening
description, a collapsed-to-sequential orchestrator) — confirm the decision propagated to *every*
file that mentions that concept, not just the one file where you made the call. This step exists
because `/engineer:review` on the first real run (Gemini) found exactly this failure mode three
times in one pass: Artifacts correctly dropped from `preferences`'s translation but still
mentioned in `warm-up` and the folded instruction file; a reworded opening sentence in one file
while a sibling file kept the untouched original; parallelism correctly collapsed in one
orchestrator's Step 1 but leftover concurrency-implying phrases surviving elsewhere in the same
file. A single correct edit in isolation is not the same as a consistent generation.

Then state what was generated and where. Explicitly — not silently — list anything that had no
clean mapping: unmatched tool names, ambiguous instructions, or target-format details this
command's own subsection didn't cover with confidence. A human should be able to act on this list
without having to re-derive it by reading the generated output themselves.

## Target-specific rules

### Gemini CLI

- **Commands** → TOML files under `.gemini/commands/<namespace>/<name>.toml` (project-level),
  namespaced by subdirectory exactly like `.claude/commands/` (`git/commit.toml` → `/git:commit`).
  Fields: `prompt`, `description`. **Confirmed** (fetched directly from Gemini CLI's own docs):
  CoDriDe's `#$ARGUMENTS` becomes `{{args}}` in the `prompt` field — injected exactly as typed
  when used in the prompt body, or automatically shell-escaped when used inside a `!{...}`
  shell-injection block. There's no positional-argument equivalent (no `$1`/`$2`-style syntax) —
  only the whole argument string, which matches CoDriDe's own convention.
- **Subagents** → `.md` + YAML frontmatter under `.gemini/agents/` (project-level, committable),
  with real `tools:`/MCP scoping.
- **Auto-loaded instructions** → root `GEMINI.md`.
- **Parallelism**: native subagent parallel dispatch exists but is experimental with known issues
  as of this writing — apply the sequential-by-default rule (step 5) unless the Gemini-specific
  branch confirms it's safe to rely on for a given orchestrator.

### Codex CLI

- **Commands** → Skills. **Confirmed** (fetched directly from `developers.openai.com/codex/skills`,
  redirecting to `learn.chatgpt.com/docs/build-skills`): a Skill is a *directory*, not a flat file —
  `.agents/skills/<skill-name>/SKILL.md`, scanned by Codex from the current working directory up to
  the repository root. Frontmatter requires `name` and `description` only (no `argument-hint` — that
  was the deprecated `~/.codex/prompts/*.md` mechanism; don't generate for it). Invocation is an
  explicit `$skill-name` mention or implicit/semantic selection when the user's request matches
  `description` — never a `/namespace:name` slash command.
- **No argument-substitution mechanism — confirmed, not just unconfirmed.** Checked directly
  against the official docs (and cross-checked a community claim of `argument-hint`/`$VARNAME`
  support, which was rejected as a likely conflation with the deprecated prompts mechanism, not the
  current one): Skills work from natural-language context only. Every source command's
  `#$ARGUMENTS` block must be **reworded**, not substituted — replace the templated-injection
  instruction with plain guidance to extract the equivalent information from the user's request
  (e.g. "the user's request should name the work item as `<type>/<slug>`; if they didn't state it
  clearly, ask before proceeding"). This is real per-file editorial judgment, not a find-replace.
- **Skill naming**: CoDriDe has real cross-namespace name collisions (`/engineer:validate` and
  `/product:validate` both exist) that a flat `name:` field can't hold. Use `<namespace>-<name>`
  (e.g. `engineer-validate`, `product-validate`) — collision-free, still traceable to the source
  namespace. The top-level `warm-up` command (no namespace in the Claude source) stays `warm-up`.
- **Subagents** → none at all, not even sequential. Every command with a fixed agent caller gets
  that agent's full logic inlined directly into its Skill body, at the point the Claude source
  invokes it (step 4's collapse rule). For the two on-demand agents with no fixed caller
  (`python-developer`, `typescript-developer`), each becomes its own standalone Skill instead of
  being inlined anywhere — an honest adaptation (Codex has no delegation concept to preserve, so
  the content survives as something the user explicitly invokes) rather than dropping them because
  the *invocation mechanism* doesn't translate.
- **Auto-loaded instructions** → a single `AGENTS.md` per directory level (first non-empty file
  wins as you walk up the tree) — no directory-of-files pattern, so step 6's concatenation is
  mandatory here, not optional.
- **Parallelism**: not applicable — there's no subagent primitive to run concurrently in the first
  place.

### GitHub Copilot CLI

- **Commands and subagents** both use the same `.agent.md` format — Copilot has no separate
  command layer, so every CoDriDe command becomes a Copilot agent. **Confirmed**: flat files at
  `.github/agents/<name>.agent.md` (project level; not a directory-per-file the way Codex is).
  Frontmatter fields: `name`, required `description`, `tools`, `model`,
  `disable-model-invocation`, `user-invocable`, `mcp-servers`. `tools` uses Copilot's own lowercase
  vocabulary (e.g. `["read", "search"]`) — sourced from a community example, not GitHub's own
  complete reference table, so treat exact tool-name mappings as reasonably confirmed but not
  fully authoritative; carry Claude's tool names over with an explicit unverified comment rather
  than guess a Copilot-native translation, same treatment already used for Gemini and Codex. Give
  a carried-over `model:` value (e.g. `opus`, `sonnet`) the same unverified-comment treatment —
  Copilot's real model identifiers weren't confirmed either, and a silently-wrong `model:` fails
  the same way a silently-wrong `tools:` entry does.
  Invocation: `/agent-name` slash command interactively, explicit natural-language instruction,
  automatic inference against `description`, or `--agent <name> --prompt "..."` programmatically.
- **`tools:` only on agents translated from `.claude/agents/`, never on agents translated from
  `.claude/commands/`.** The 8 core agents each declare a real `tools:` allowlist in their Claude
  source frontmatter — carry that over (with the unverified-comment treatment above). Commands
  declare no such thing on Claude Code (they run unrestricted), so don't invent a `tools:`
  allowlist for the 27 command-derived agents — omit the field entirely, matching the source. A
  guessed allowlist on a command is worse than none: it silently narrows what an orchestrator like
  `engineer-pre-pr` (which exists purely to delegate) or a discovery command that shells out can
  actually do, in a way nothing in the source authorizes.
- **No argument-substitution mechanism found** — same absence as Codex, not re-derived: reword
  every `#$ARGUMENTS` block into natural-language extraction guidance, don't substitute.
- **No collapse needed — unlike Codex.** Copilot has real subagent delegation (the model can
  autonomously delegate to a custom agent, and v1.0.66+ usage-based-billing plans can configure
  concurrency and delegation-depth limits explicitly). Every command keeps its own
  `user-invocable: true` agent file and delegates to the appropriate core agent's
  `user-invocable: false` file, exactly as the Claude source's command→agent relationship already
  works — just re-expressed in Copilot's single unified format instead of two separate ones.
  **Exception**: `python-developer` and `typescript-developer` are the two core agents with no
  fixed calling command on Claude Code — CLAUDE.md's own agent table documents them as used "on
  demand," i.e. invoked directly by a human, not delegated to by another command. Give both
  `user-invocable: true` here, same carve-out already made for these two on the Codex target
  (there, they became standalone directly-invocable Skills for the same reason). The other 6 core
  agents stay `user-invocable: false`.
- **Auto-loaded instructions** → generate a dedicated `.github/copilot-instructions.md` (folded,
  same as Gemini/Codex) — don't rely on Copilot's confirmed ability to also directly read
  `AGENTS.md`/`CLAUDE.md`/`GEMINI.md` if present (it does, and combines/deduplicates them with no
  defined precedence order), since a Copilot-only adopter never receives those other targets'
  files. Document the auto-read as a bonus in the guide, not a mechanism to depend on. Copilot also
  supports a real directory-of-files pattern (`.github/instructions/**/*.instructions.md`, each
  requiring an `applyTo` glob) — not used by this generator, since a single folded file already
  satisfies step 6, but worth knowing it exists if a future need for path-scoped instructions
  arises.
- **Parallelism**: real and configurable, but plan-gated (not available to every adopter by
  default) — apply the sequential-by-default rule (step 5) for the generated output regardless;
  note in that target's guide that adopters on a qualifying plan could enable it themselves.
