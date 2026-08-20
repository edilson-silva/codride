---
description: Turn context.md + architecture.md into a phased plan.md
argument-hint: <type>/<slug>
---

# Engineer Plan

This is the command to start planning a feature's execution.

<arguments>
#$ARGUMENTS
</arguments>

## Review

Read `context.md` and `architecture.md` in `.claude/work/<type>/<slug>` if you haven't already. If either file doesn't exist yet, stop and tell the human to run `/engineer:context` and/or `/engineer:architecture` first — don't invent a plan without them.

Your task now is to create a detailed execution plan, saved to `.claude/work/<type>/<slug>/plan.md`. The goal of this document is a phased execution approach that lets us build the feature incrementally, testing each phase as we go, and that makes it possible to resume work if you get interrupted.

`plan.md` should split execution into phases, each phase sized to a chunk of work a human could complete in about 2 hours.

Template for `plan.md`:

<plan>
# [WORK NAME]

If you're working on this, keep this plan.md file updated as you progress.

## PHASE 1 [Done ✅]

Details of this part of the feature.

### A task that was done [Done ✅]

Details about the task.

### A task that was done [Done ✅]

Details about the task.

### Notes:
- Something that happened and forced a change of direction.
- Something we learned while building.
- Something we discussed and agreed on.

## PHASE 2 [In Progress ⏰]

### A task that needs doing [In Progress ⏰]

Details about the task.

### A task that needs doing [Not Started ⏳]

Details about the task.

## PHASE 3 [Not Started ⏳]

### A task that needs doing [Not Started ⏳]

Details about the task.

### A task that needs doing [Not Started ⏳]

Details about the task.

</plan>

Tips:
- Use `Glob`/`Grep` to find specific files based on your discovery answers, and `Read` to batch through relevant code.
- Examine specific implementation details.
- Use WebSearch (and Context7, if available) for best practices or library documentation when needed.

If this research surfaces a new architectural decision or a contradiction with earlier decisions, start a discussion about it with the human, agree on the changes, and update `architecture.md` for this feature if needed. If the decision is significant enough to constrain future work beyond this feature, suggest recording it with `/engineer:adr`.

This artifact should also note which tasks need to happen sequentially versus which can run in parallel.

Once `plan.md` is finalized, tell the human you're ready to move to the next step.

⛔ **Don't proceed automatically. Wait for the human to approve the plan and explicitly run `/engineer:work`. Don't start implementation on your own.**
