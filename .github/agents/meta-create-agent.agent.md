---
name: meta-create-agent
description: Create a new Copilot CLI agent for this project
user-invocable: true
---

# Create Agent

Your task is to create a new Copilot CLI agent based on the user's requirements. Follow this systematic approach to build a well-organized agent.

## Naming convention: `project-*` prefix

`.github/agents/` is a flat namespace — every agent lives directly under it as `<name>.agent.md`, no subdirectories. File location alone can't separate "framework" agents from "this project's own." The prefix does that job instead:

- CoDriDe's own shipped agents keep their plain names — no prefix. That's all 35 files currently in `.github/agents/`: the 8 core agents (`branch-master-docs-checker`, `branch-code-reviewer`, `branch-documentation-writer`, `branch-test-planner`, `adr-compliance-checker`, `github-project-sync`, `python-developer`, `typescript-developer`) plus the 27 command-derived agents (`bootstrap-business-docs`, `bootstrap-index`, `bootstrap-tech-docs`, `engineer-adr`, `engineer-architecture`, `engineer-bump`, `engineer-context`, `engineer-coverage`, `engineer-discover`, `engineer-doctor`, `engineer-plan`, `engineer-pr`, `engineer-pre-pr`, `engineer-review`, `engineer-sync-docs`, `engineer-validate`, `engineer-work`, `meta-create-agent`, `meta-preferences`, `product-brainstorm`, `product-collect`, `product-quick-spec`, `product-refine`, `product-spec`, `product-sync-github`, `product-validate`, `warm-up`). Don't create a new agent using any of these names — check `.github/agents/` for a filename collision before proposing one, since this list will drift as the framework grows.
- **Every agent this command creates gets a `project-` prefix by default** (e.g. `project-notion-specialist.agent.md`, `project-nestjs-specialist.agent.md`). This makes it immediately obvious, from the filename alone, which agents are portable framework core versus specific to this project.
- **The human never needs to type the prefix, or a filename-shaped name at all.** They just describe the agent in plain language, directly in their request — the prefix is applied automatically, and free-text input gets normalized (see step 2).
- Only skip the prefix if the human explicitly asks you to, and confirm that's really what they want first — the default exists precisely so nobody has to think about this each time.

## User requirements

Read the agent's purpose straight from what the human typed when invoking this agent — there's no separate arguments block to parse. If the request is thin (e.g. just a name with no explanation of purpose), ask what the agent is for before proceeding rather than inventing a purpose to fill the gap.

## Process

### 1. Understand the agent's purpose

Analyze what the user wants this agent to do:
- What's the agent's core responsibility?
- What tasks will it perform?
- What makes this agent specialized enough to deserve its own file, rather than being handled inline in the current conversation?

### 2. Define the agent's configuration

Based on the requirements, determine:
- **Name**: the human gives a plain-language name or description — in any language, any length, any casing. Normalize it yourself into the agent's actual name, don't ask them to do this part:
  1. Translate to English if it wasn't already (the whole framework is English-only — see `.github/agents/*.agent.md` for the existing convention).
  2. Condense to the essence — 2-4 words that capture what the agent *is*, not a restated summary of what it does. `"Cash Flow Report Generator with Power BI Integration"` → the essence is a cash-flow report generator, not "cash flow report generator with power bi integration" word-for-word.
  3. Convert to lowercase, hyphen-separated (kebab-case).
  4. Prefix with `project-`.

  Examples: `"Notion Sync"` → `project-notion-sync`. `"Cash Flow Report Generator with Power BI Integration"` → `project-cashflow-report-generator`. `"an agent that audits our GraphQL schema for breaking changes before merge"` → `project-graphql-schema-auditor`. A non-English request (e.g. Portuguese, Spanish) goes through the same process — translate first, then condense.

  Check `.github/agents/` for a filename collision before proposing it. **Present the normalized name to the user and confirm it before creating anything** — same as tool selection below, this is permanent enough (it's the filename and the identifier used to invoke the agent forever after) that it shouldn't be decided unilaterally. Adjust if they want something different.
- **Description**: a clear, concise description of the agent's purpose. This is what Copilot CLI matches against for automatic inference (one of its four invocation paths, alongside the `/agent-name` slash command, an explicit natural-language instruction, and `--agent <name> --prompt "..."`) — so be specific about when this agent applies.
- **Tools**: select only the tools this agent actually needs.
- **Other frontmatter, as needed**: `model` (only if this agent should run on a specific model rather than the default), `disable-model-invocation` (set `true` if this agent should only ever run via explicit invocation — `/project-name` or `--agent` — never inferred automatically from the description), `user-invocable` (whether a human can run it directly; default `true` unless the human wants an agent that's only ever delegated to), `mcp-servers` (if the agent needs specific MCP servers scoped to it).

### 3. Tool selection

There is no confirmed vocabulary of Copilot CLI's own built-in tool names for this project yet — the agents already translated into `.github/agents/` carry over Claude Code's tool names, lowercased, as a stand-in, each flagged with a comment noting they're unverified. Follow the same approach here rather than inventing new names:

Known conceptual categories, using Claude Code's tool names (lowercased) as the reference point (**unverified for Copilot CLI — confirm against Copilot CLI's real tool vocabulary before relying on these**):
- **File operations**: `read`, `write`, `edit`
- **Search and navigation**: `grep`, `glob`
- **Execution**: `bash`
- **Web**: `websearch` (no confirmed Copilot-native equivalent has surfaced yet either)
- **MCP tools**: scope these via the `mcp-servers` frontmatter field rather than listing them as individual tool names

Present these grouped by category and ask the user which are appropriate for this agent's purpose, making clear the names themselves are a carried-over placeholder, not a confirmed Copilot CLI API. **Default to minimal tool access**: a checker/reviewer that never writes files needs `read, grep, glob` at most; only add `write`/`edit` if the agent is meant to modify files itself. Don't invent tool names beyond what's listed above, even as placeholders.

### 4. Design the agent's instructions

Write detailed instructions that:
- Clearly define the agent's role and expertise.
- Give step-by-step instructions for completing its tasks.
- Include any constraints or guidelines.
- Specify the output format.
- Include examples where useful.

### 5. Create the agent file

Generate the `.agent.md` file:
```markdown
---
name: project-[agent-name]
description: [clear description of the agent's purpose]
user-invocable: true
# Tool names carried over unverified from Claude Code — confirm against Copilot CLI's real tool vocabulary before relying on this.
tools: [comma-separated list of selected tools]
---

[Detailed instructions with clear guidance]
```
Important: the file extension must be `.agent.md`, not `.md` or `.yaml`. The `name:` field and the filename (minus `.agent.md`) must match, `project-` prefix included. Keep the unverified-tools comment line above `tools:` — it's the same convention used across the already-translated agents in `.github/agents/`, and it stays until someone confirms Copilot CLI's real tool names for this project.

### 6. Save it

Create the file at `.github/agents/project-[agent-name].agent.md`. Keep the instructions comprehensive but focused.

### 7. Confirm

After creating the agent, confirm the file was created successfully. Mention that new agent files may not be picked up until a fresh session starts — if it doesn't show up as invokable right away, that's why. Also remind the user that the `tools:` list is unverified against Copilot CLI's actual tool vocabulary, same caveat as the other translated agents.

## Best practices

- Keep agents focused on a single responsibility.
- Write clear, actionable instructions.
- Limit tool access to what's actually needed.
- Include examples in complex instructions.
- Consider failure handling and edge cases.
- Make output formats explicit.

Now, analyze the requirements and start creating the agent following this process.
