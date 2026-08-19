---
description: Sync docs/business-context/features/ with GitHub Issues
argument-hint: [module=<name> | preview]
---

# Sync Features with GitHub

Syncs the feature documentation in `docs/business-context/features/` with this repo's GitHub Issues, creating or updating parent issues and sub-issues automatically.

## Flow

### Step 1: Pick a mode

Ask the user which sync mode they want (use `AskUserQuestion`):

1. **Full** — sync every feature from every module.
2. **By module** — sync only one module's features.
3. **Preview** — only produce the diff report, no changes.

### Step 2: Analyze features

Invoke the `github-project-sync` agent to run the analysis: it reads `docs/business-context/features/*.md` (skipping `_legacy*`, `index.md`, `README.md`), extracts each feature's title, status, priority, scope, module, and requirement sections (`FR-XX`), and compares them against this repo's existing GitHub Issues (`gh issue list --state all`).

### Step 3: Review the preview report

The agent presents a diff report — features to create, features to update, features already in sync, and orphaned issues (tagged as synced but no longer matching a feature file). Review it with the user before anything is written.

### Step 4: Confirm

Ask the user how to proceed: run everything (create + update), only create, only update, or cancel.

### Step 5: Execute

If approved, the agent creates/updates issues via `gh issue create` / `gh issue edit` / `gh issue close`, following the field mapping and safety rules documented in the `github-project-sync` agent (never duplicate, never silently downgrade status, always preview before writing, never overwrite a manually-edited body).

### Step 6: Final report

Present what was created, updated, and skipped, with issue URLs, and suggest next steps (review the created issues, assign owners, add estimates if needed).

## Usage

```
/product:sync-github
/product:sync-github module=Billing
/product:sync-github preview
```
