---
description: Create a new Claude Code sub-agent for this project
argument-hint: <agent requirements>
---

# Create Agent Command

Your task is to create a new Claude Code sub-agent based on the user's requirements. Follow this systematic approach to build a well-organized agent.

## Naming convention: `project-*` prefix

`.claude/agents/` is a flat namespace — Claude Code doesn't discover agents in subdirectories, so file location can't separate "framework" agents from "this project's own." The prefix does that job instead:

- Codride's 8 shipped agents (`branch-master-docs-checker`, `branch-code-reviewer`, `branch-documentation-writer`, `branch-test-planner`, `adr-compliance-checker`, `github-project-sync`, `python-developer`, `typescript-developer`) keep their plain names — no prefix. Don't create a new agent using one of these names.
- **Every agent this command creates gets a `project-` prefix by default** (e.g. `project-notion-specialist.md`, `project-nestjs-specialist.md`). This makes it immediately obvious, from the filename alone, which agents are portable framework core versus specific to this project.
- **The human never needs to type the prefix, or a filename-shaped name at all.** They just describe the agent in plain language — the prefix is applied automatically, and free-text input gets normalized (see step 2).
- Only skip the prefix if the human explicitly asks you to, and confirm that's really what they want first — the default exists precisely so nobody has to think about this each time.

## User requirements
<requirements>
#$ARGUMENTS
</requirements>

## Process

### 1. Understand the agent's purpose

Analyze what the user wants this agent to do:
- What's the agent's core responsibility?
- What tasks will it perform?
- What makes this agent specialized enough to deserve its own file, rather than being handled inline by the main agent?

### 2. Define the agent's configuration

Based on the requirements, determine:
- **Name**: the human gives a plain-language name or description — in any language, any length, any casing. Normalize it yourself into the agent's actual name, don't ask them to do this part:
  1. Translate to English if it wasn't already (the whole framework is English-only — see `.claude/agents/*.md` for the existing convention).
  2. Condense to the essence — 2-4 words that capture what the agent *is*, not a restated summary of what it does. `"Cash Flow Report Generator with Power BI Integration"` → the essence is a cash-flow report generator, not "cash flow report generator with power bi integration" word-for-word.
  3. Convert to lowercase, hyphen-separated (kebab-case).
  4. Prefix with `project-`.

  Examples: `"Notion Sync"` → `project-notion-sync`. `"Cash Flow Report Generator with Power BI Integration"` → `project-cashflow-report-generator`. `"an agent that audits our GraphQL schema for breaking changes before merge"` → `project-graphql-schema-auditor`. A non-English request (e.g. Portuguese, Spanish) goes through the same process — translate first, then condense.

  Check `.claude/agents/` for a filename collision before proposing it. **Present the normalized name to the user and confirm it before creating anything** — same as tool selection below, this is permanent enough (it's the filename and the identifier used to invoke the agent forever after) that it shouldn't be decided unilaterally. Adjust if they want something different.
- **Description**: a clear, concise description of the agent's purpose — this is what the main agent uses to decide when to invoke it, so be specific about when it applies.
- **Tools**: select only the tools this agent actually needs.

### 3. Tool selection

Real Claude Code tool names to choose from:
- **File operations**: `Read`, `Write`, `Edit`, `NotebookEdit`
- **Search and navigation**: `Glob`, `Grep`
- **Execution**: `Bash`, `Agent` (to delegate to other sub-agents)
- **Web**: `WebFetch`, `WebSearch`
- **Other**: `TodoWrite`, `AskUserQuestion`
- **MCP tools**: any `mcp__<server>__<tool>` tools enabled in this project

Present these grouped by category and ask the user which are appropriate for this agent's purpose. **Default to minimal tool access**: a checker/reviewer that never writes files needs `Read, Glob, Grep, Bash` at most; only add `Write`/`Edit` if the agent is meant to modify files itself. Don't invent tool names that aren't in the list above.

### 4. Design the system prompt

Write a detailed system prompt that:
- Clearly defines the agent's role and expertise.
- Gives step-by-step instructions for completing its tasks.
- Includes any constraints or guidelines.
- Specifies the output format.
- Includes examples where useful.

### 5. Create the agent file

Generate the `.md` file:
```markdown
---
name: project-[agent-name]
description: [clear description of the agent's purpose]
tools: [comma-separated list of selected tools]
---

[Detailed system prompt with clear instructions]
```
Important: the file extension must be `.md`, not `.yaml`. The `name:` field and the filename must match, `project-` prefix included.

### 6. Save it

Create the file at `.claude/agents/project-[agent-name].md`. Keep the system prompt comprehensive but focused.

### 7. Confirm

After creating the agent, confirm the file was created successfully.

## Best practices

- Keep agents focused on a single responsibility.
- Write clear, actionable system prompts.
- Limit tool access to what's actually needed.
- Include examples in complex prompts.
- Consider failure handling and edge cases.
- Make output formats explicit.

Now, analyze the requirements and start creating the agent following this process.
