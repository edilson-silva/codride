---
description: Scan the codebase and generate the technical-context briefing
argument-hint: [--verbose]
---

# Project Discovery & Context Mapping

This command analyzes the project automatically and generates a complete technical briefing in a modular format.

It is **optional** and can be run:
- Once, at the start of the project
- When ADRs change significantly
- When the backend architecture changes

## Incremental behavior

This command is **non-destructive**:
- If briefing files already exist: update them incrementally
- If they don't exist: create them from scratch
- Always preserve existing information

## Generated structure

```
docs/technical-context/
├── project-briefing.md              # Master index + summary
└── briefing/
    ├── critical-rules.md            # Non-negotiable rules
    ├── adrs-summary.md              # Consolidated ADRs
    ├── backend-conventions.md       # Code conventions
    └── tech-stack.md                # Tech stack
```

---

## Phase 1: ADR analysis (conditional)

### 1.1 Detect ADRs

```bash
paths_to_check = [
  "docs/technical-context/adr/",
  "docs/adr/",
  "adr/"
]
```

- If **none** of these paths exist: skip this phase entirely.
- If one exists: proceed with the analysis.

### 1.2 Process ADRs

For each `.md` file found in the ADR folder:

1. **Read the full file.**
2. **Extract:**
   - ADR number (e.g. ADR-001, ADR-015)
   - Title
   - Status (Accepted, Proposed, Rejected, Superseded)
   - Context/background
   - Core decision
   - Mandatory conventions (if any)
   - Consequences
   - Category (infer: database, API, code organization, security, etc.)
   - Impact (high/medium/low — infer from criticality)
3. **Identify critical rules**: look for keywords like "mandatory", "always", "never", "forbidden". Flag high-impact ADRs.

### 1.3 Consolidate ADRs

- Group by category (database, API, code organization, testing, security, etc.)
- Order by impact (high → medium → low)
- Build an index with links

### 1.4 Check existing code against the consolidated ADRs

Invoke the `adr-compliance-checker` agent against the current codebase, now that the ADRs are consolidated. This matters most when onboarding an existing project — ADRs are often written after the fact, once a convention has already drifted, so this is the first chance to see where the code and the documented decisions disagree.

If Phase 2.0 below determines this is a monorepo, don't run this as a single generic pass — invoke `adr-compliance-checker` once per service found in 2.0, the same way Phases 2.1-2.4 do, since a convention violation in `apps/api/` may not apply to `apps/worker/` at all.

Advisory only: collect the findings for the Phase 4.2 report; don't block Phase 2-4 on them, and don't fix anything automatically.

---

## Phase 2: Backend architecture analysis (always runs)

### 2.0 Detect single-project vs. monorepo

Check for monorepo signals at the root: a `package.json` with a `workspaces` field, `pnpm-workspace.yaml`, `lerna.json`, `nx.json`, `turbo.json`, or multiple independent manifest files under `apps/*/` or `packages/*/`.

- **If none of these exist**: single project. Continue with 2.1 below as normal.
- **If they do**: this is a monorepo. List every workspace/app/package that has its own manifest file — don't just take the first match and stop. Run 2.1–2.4 **for each one that looks like a backend/service** (has its own manifest and server-side code, not just a shared UI-component or config package). Keep each service's findings separate; don't merge them into one generic answer. A monorepo with `apps/api/`, `apps/worker/`, and `packages/shared/` gets three sub-analyses, not one.

### 2.1 Detect the backend / main codebase

Look for common paths:
```bash
backend_paths = [
  "apps/backend/", "backend/", "server/", "api/",
  "packages/backend/", "src/backend/", "src/"
]
```

In monorepo mode, apply this per workspace found in 2.0 instead of at the repo root.

### 2.2 Map the folder structure

For the codebase found, scan for:
- Controllers/routes → HTTP handlers
- Services → business logic
- Repositories/DAL → data access
- Models/entities → domain models
- Shared code (enums, types, validation schemas, utilities)
- Tests

### 2.3 Identify conventions

**Shared type/enum location**: find where the majority of enum/type definitions live; treat it as a convention if >80% share the same location. In monorepo mode, also check whether a `packages/shared/`-style package is where cross-service types actually live — that's itself a convention worth recording.

**Architectural patterns**: repository pattern? Service layer? MVC? — infer from which folders coexist.

**Naming**: sample 5-10 files to identify the dominant convention (kebab-case vs. camelCase vs. PascalCase, suffix conventions like `.controller.ts`/`.service.ts` or `_service.py`).

### 2.4 Detect the tech stack

Read the manifest file (`package.json`, `pyproject.toml`, etc.) — per service, in monorepo mode, since different services in the same monorepo commonly run different stacks:
- Web framework (Express, Fastify, NestJS, Django, FastAPI, Flask, ...)
- Database/ORM (Prisma, TypeORM, Drizzle, SQLAlchemy, Mongoose, ...)
- Validation library (Zod, Joi, Pydantic, class-validator, ...)
- Test framework (Jest, Vitest, pytest, ...)
- Runtime/language version

---

## Phase 3: Generate the briefing files

### 3.1 Create the folder if it doesn't exist

```bash
mkdir -p docs/technical-context/briefing/
```

### 3.2 `project-briefing.md` (master index)

Include: an executive-summary project status, an index of the briefing files, a usage guide by feature type, and maintenance instructions. Target size: ~150 lines.

### 3.3 `briefing/critical-rules.md`

The 3-5 most critical rules from the ADRs, mandatory conventions, and a compliance checklist. **This file gets copied in full into every `context.md`.** Target size: ~80 lines.

### 3.4 `briefing/adrs-summary.md`

If ADRs exist: an index by category, each ADR summarized with title, status, impact, decision, mandatory conventions, and a link to the full ADR. If no ADRs exist: create the file anyway with a note that none are defined yet. Target size: scales with ADR count.

### 3.5 `briefing/backend-conventions.md`

Folder structure (ASCII tree), file/class/method naming, code patterns (controller/service/repository or equivalent), and a couple of representative code examples. In monorepo mode, use one subsection per service instead of collapsing them into a single generic answer — a convention that holds in `apps/api/` may not hold in `apps/worker/`. Target size: ~150 lines (scales with the number of services).

### 3.6 `briefing/tech-stack.md`

Runtime/language version, web framework, database + ORM, key libraries, and important version constraints. In monorepo mode, one subsection per service, plus a line noting which package manager/workspace tool ties them together (pnpm workspaces, Nx, Turborepo, Lerna). Target size: ~100 lines (scales with the number of services).

---

## Phase 4: Validation and wrap-up

### 4.1 Sanity check

- All files created successfully?
- Do cross-links between files resolve?
- No unexpectedly empty file?

### 4.2 Report to the human

```
✅ Project Discovery complete.

📁 Files generated:
- docs/technical-context/project-briefing.md (master index)
- docs/technical-context/briefing/critical-rules.md
- docs/technical-context/briefing/adrs-summary.md
- docs/technical-context/briefing/backend-conventions.md
- docs/technical-context/briefing/tech-stack.md

📊 Summary:
- ADRs analyzed: X
- Critical rules: X
- Backend framework(s): [name, or one per service in monorepo mode]
- Database/ORM: [name]
- ADR compliance (adr-compliance-checker): [no ADRs to check / N violations found, listed below / clean — one line per service in monorepo mode]

[If violations were found, list each: which ADR, where in the code, and the suggested fix.]

💡 Next steps:
1. Review project-briefing.md for accuracy.
2. Review critical-rules.md (it gets copied into every context.md).
3. Run /engineer:context when ready to start a feature.

ℹ️  This briefing is loaded automatically by /engineer:context to enrich context.
🔄 To update it, run /engineer:discover again (incremental, non-destructive).
```

⛔ **After reporting to the human, STOP. Don't automatically proceed to `/engineer:context` or any other command. Wait for feedback.**

---

## Tools used

- **Glob**: to find files (ADRs, manifest files, source files).
- **Read**: to read file contents.
- **Write**: to create/update the briefing files.

---

## Error handling

- **Malformed ADR**: warn the human ("ADR X is malformed, skipping"), continue with the others, and list what was skipped at the end.
- **Unrecognized backend structure**: ask the human where controllers/routes, services, and shared enums/types live, instead of guessing.
- **No manifest file found**: warn that analysis will be limited and document what could be inferred from the folder structure alone.

---

## Verbose mode (optional)

`/engineer:discover --verbose` shows detailed progress (each ADR processed, each file scanned, each generated file). The default is a summarized progress report with only warnings/errors shown.

---

## Golden rules

1. **Never assume** — if something can't be found, ask the human.
2. **Be incremental** — don't overwrite existing information without reason.
3. **Be explicit** — cross-links between files must resolve.
4. **Be resilient** — an error in one phase shouldn't stop the whole process.
