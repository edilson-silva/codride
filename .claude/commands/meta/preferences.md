---
description: Configure project-level preferences (documentation language, Artifacts publishing) — extensible for more later
---

# Preferences

This configures how Claude behaves **around this project's written artifacts** — not the project's business/product content itself, and not conversational behavior. It doesn't touch code, master docs, or `.claude/agents/`. Safe to re-run anytime to change an answer.

Always project-scoped — this isn't a question about how you want Claude configured across every project on your machine, just this one. The answers live in `docs/PROJECT_PREFERENCES.md`, versioned alongside the rest of the project's docs, so the whole team gets the same behavior, not just whoever ran this command.

## 1. Check current state

Before asking anything, read `docs/PROJECT_PREFERENCES.md` if it exists, and check `.claude/settings.local.json` for an existing `permissions.deny` entry covering the `Artifact` tool.

Report what's already set before asking about it. Only ask about what's missing, or what the human explicitly says they want to revisit — never re-run the full interview from scratch just because the command was invoked again.

## 2. Ask

### Documentation language
"What language should `docs/` and `.claude/work/` artifacts be written in — `context.md`, `architecture.md`, `plan.md`, business-context files, ADRs, and so on?"

Be explicit this is **not** about conversation: Claude already mirrors whatever language the human writes in, session to session, with no configuration needed. This setting only controls the language of files that get committed to the repo.

### Artifacts
"Can Claude publish Artifacts (claude.ai-hosted pages) for this project?"

If no: this gets enforced, not just documented (see step 3) — publishing goes through the `Artifact` tool, which Claude Code's own permission system can block outright.

## 3. Write

- **`docs/PROJECT_PREFERENCES.md`**: create it if it doesn't exist, or update the relevant section if it does (don't touch sections the human didn't just answer). Keep it short and plain:

  ```markdown
  # Project Preferences

  ## Documentation language
  Write `docs/` and `.claude/work/` artifacts in: <language>. Conversational
  responses aren't affected — Claude already mirrors whatever language the
  human writes in.

  ## Artifacts (claude.ai-hosted pages)
  <Allowed | Not allowed — enforced via `.claude/settings.local.json`'s
  `permissions.deny`, see below>
  ```

- **`.claude/settings.local.json`**: only if Artifacts were denied. Add `"Artifact"` to `permissions.deny` (create the key, or the whole file, if it doesn't exist yet — merge into existing content, don't overwrite the `permissions.allow` array already there). Skip if the rule is already present. If the human is re-enabling Artifacts on a re-run, remove the rule instead — confirm first, don't delete silently.

  Note out loud that this file is machine-local and gitignored by convention (see `README.md`'s "Configuration File" section) — a teammate on a different machine needs to run `/meta:preferences` themselves (or add the rule by hand) to get the same hard enforcement locally, even though `docs/PROJECT_PREFERENCES.md` already tells them the project's intent.

## 4. Confirm

```markdown
## Preferences set
- Documentation language: [English (default) / <language>]
- Artifacts: [allowed / denied]

## Where these live
- docs/PROJECT_PREFERENCES.md — the record every `/warm-up` reads
- .claude/settings.local.json — the actual enforcement, if Artifacts were denied (machine-local)
```

Re-run `/meta:preferences` anytime to change an answer.
