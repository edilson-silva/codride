---
description: Start a unit of work: interview to build shared understanding, written to context.md
argument-hint: <type>/<slug>
---

# Work Context

This command kicks off a new unit of work — a feature, fix, chore, docs update, or anything else that will get its own work item — by building shared understanding of what's being built and writing it to `context.md`. Everything below says "feature" for readability, but applies the same way to any type of work.

Work items live under `.claude/work/<type>/<slug>/`, where `<type>` is a [Conventional Commits](https://www.conventionalcommits.org/) type (`feat`, `fix`, `docs`, `chore`, `refactor`, `test`, `perf`, `build`, `ci`, `style`, `revert`) and `<slug>` is a short kebab-case name — e.g. `.claude/work/fix/password-reset-plus-alias/`. This mirrors typical branch naming (`fix/password-reset-plus-alias`), so the work item folder and the branch name usually match.

This is the first of two steps: `/engineer:context` (this one) writes `context.md`; `/engineer:architecture` then designs the implementation and writes `architecture.md`, cross-checked against this file.

## Context enhancement (optional — if a project briefing exists)

**Before asking the user for input, run this check:**

### 1. Check for a project briefing

```bash
if exists("docs/technical-context/project-briefing.md"):
  load_briefing = true
else:
  load_briefing = false
  # Continue with the original flow (no enhancement)
```

### 2. Load essential context (if the briefing exists)

**If `load_briefing = true`:**

1. **Always read** (small, essential files):
   - `docs/technical-context/project-briefing.md` (master index, ~150 lines)
   - `docs/technical-context/briefing/critical-rules.md` (mandatory rules, ~80 lines)

2. **Ask the engineer** (to know what else to load):
   ```
   To optimize the context for this feature, I need to know which backend
   areas are impacted (check all that apply):

   [ ] API/Controllers (creating or modifying endpoints)
   [ ] Database/ORM (schema or query changes)
   [ ] Authentication/Authorization
   [ ] Services/Business logic
   [ ] Other: ___________

   This lets me load only the ADRs and conventions relevant to this
   feature, saving tokens and staying focused on what matters here.
   ```

3. **Load selectively based on the answers:**
   - Always load `briefing/backend-conventions.md` if any area is checked.
   - If "API/Controllers" is checked: read the "API Design" section of `briefing/adrs-summary.md`.
   - If "Database/ORM" is checked: read the "Database & Persistence" section.
   - If "Authentication" is checked: read the "Security" section.

### 3. Enrich context.md with the briefing

**When generating `context.md`, always include at the top (if a briefing exists):**

```markdown
# Context: [Feature Name]

## ⚠️ Critical Project Rules (from the Project Briefing)

[PASTE the full content of briefing/critical-rules.md here]

## 📚 Relevant ADRs (quick reference)

[List the ADRs loaded, with links]

- ADR-003: Shared Types Location → `docs/technical-context/briefing/adrs-summary.md#adr-003`
- ADR-007: Repository Pattern → `docs/technical-context/briefing/adrs-summary.md#adr-007`

---

## Feature-Specific Context

[Rest of context.md as usual...]
```

**Result**: `context.md` ends up ~300-400 lines (critical rules + feature-specific context) instead of relying on memory or links that can be forgotten.

**If `load_briefing = false`** (no briefing exists): skip this whole section and continue to "Preparation" below — original flow, unchanged.

---

## Preparation

- If you're not on a branch for this work, ask for authorization to create one, named `<type>/<slug>` to match the work item folder.
- If you're already on a branch matching this work item's slug, you're ready.
- Check whether `.claude/work/<type>/<slug>` already exists. If `context.md` is already there, confirm with the human whether you're revising it or this is a duplicate of existing work — don't silently overwrite it.
- Ask the user for this work item's input (you'll receive one or more GitHub issues to work from).

## Discovery

Go through the issues — and their parent/child issues if relevant — and build an initial understanding of what needs to be built. Think carefully about what's being asked, making sure you understand exactly:
- Why this is being built (context)
- What the expected outcome is for this issue (goal)
- How it should be built, directionally only, not in detail (approach)
- Whether it requires new APIs/tools, and whether you understand them
- How it should be tested
- What the dependencies are
- What the constraints are

If the source issue has acceptance criteria (`Given/When/Then`, written by `/product:spec` or `/product:quick-spec`), carry them into `context.md` verbatim — don't paraphrase or summarize them away. `/engineer:work` implements test-first against them, and `/engineer:coverage` checks they're all covered, so they need to survive intact.

After thinking through these questions, formulate the 3-5 most critical clarifications needed to complete the task. Present these to the human along with your understanding and proposals.

After getting the human's answers, decide whether you need further clarification. If so, keep the dialogue going.

Once you have a solid understanding of what's being built, save it to `.claude/work/<type>/<slug>/context.md` and ask the human to review it.

If the human agrees with your understanding, tell them `/engineer:architecture <type>/<slug>` is the next step. Otherwise, keep iterating together until you get explicit approval.

If anything discussed here affects what's documented in the requirements, ask the human for permission to edit those requirements — either by editing them directly (structural changes) or adding comments (clarifications). If the requirement lives in a GitHub issue, update the issue; if it's a text file, update the file.

⛔ **Don't proceed automatically to architecture design. Wait for the human's explicit approval and for them to invoke `/engineer:architecture` manually.**

<slug>
#$ARGUMENTS
</slug>

Expected format: `<type>/<slug>`, e.g. `feat/csv-order-export` or `fix/password-reset-plus-alias`.
