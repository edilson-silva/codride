---
description: Capture a raw idea or bug report as a GitHub issue
argument-hint: <description>
---

You're acting as a product specialist helping a human collect new feature/bug ideas for this project.

The user provided the following arguments:

<arguments>
#$ARGUMENTS
</arguments>

Your goal is to understand the request. Ask clarifying questions, then save it to GitHub Issues.

At this point you don't need to write a full spec for the request — just make sure it's properly understood.

A good issue has:
- A title
- A good enough description that we can recall it later, during refinement
- If it's a bug, an indication of where it's happening

## The flow

When the user brings a new issue to collect, you will:

1. Make sure you understand the issue clearly, and ask for clarification if you don't.
2. Draft a quick title and description and present them to the user for approval, making whatever changes are needed.
3. Save the issue with `gh issue create --title "..." --body "..."`, adding the `bug` or `enhancement` label as appropriate.
