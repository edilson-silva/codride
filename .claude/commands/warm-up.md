---
description: Load product + engineering context before starting work
argument-hint: [project-name]
---

# Warm-up

Load both the product and engineering context for this project before starting work — not everything, just the entry points, so you know what exists and can pull in the rest later as the need arises.

## 1. Product context (`docs/business-context/`)

Read `docs/business-context/index.md` if it exists. Keep in mind what it points to (product strategy, customer personas, competitive landscape, feature catalog, etc.) without reading those files yet.

If `docs/business-context/` doesn't exist yet, say so and skip this step — it can be generated with `/bootstrap:business-docs`.

## 2. Engineering context (`docs/technical-context/`)

This folder can take two different shapes depending on which command generated it — read whichever of these exists:
- `docs/technical-context/project-briefing.md` — from `/engineer:discover`. Compact (~150 lines), meant to be read in full.
- `docs/technical-context/index.md` — from `/bootstrap:tech-docs`. Points to a fuller set (project charter, ADRs, codebase guide, business logic, API spec, etc.) — read the index, not everything it links to.

Keep in mind what other files it points to (ADRs, conventions, tech stack) without reading those yet either.

If `docs/technical-context/` doesn't exist yet, say so and skip this step — it can be generated with `/engineer:discover` (lightweight) or `/bootstrap:tech-docs` (comprehensive).

## 3. Project overview

Read the root `README.md`. Keep a list of the files under `docs/` in context so you can refer to them later as needed, without reading all of them up front.

Project name (optional): #$ARGUMENTS
