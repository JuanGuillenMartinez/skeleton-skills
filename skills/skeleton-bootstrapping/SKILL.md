---
name: skeleton-bootstrapping
description: >
  Use at the start of every conversation, when receiving any task request.
  Loads project-specific context (CLAUDE.md + ARCHITECTURE.md) and routes
  the user's intent to the appropriate skeleton:* skill. Replaces
  using-superpowers as the entry point for this project.
---

# Skeleton Bootstrapping

Entry point for every session. Loads architecture context and routes intent before any work begins.

**Announce:** "Using skeleton:bootstrapping to load project context and determine the workflow."

## Priority Hierarchy

When instructions conflict, highest priority wins:

| # | Source | Example |
|---|--------|---------|
| 1 | CLAUDE.md (per project) | "No SoftDeletes" overrides laravel-specialist |
| 2 | ARCHITECTURE.md (per project) | `Modules/` structure overrides `App\Models` |
| 3 | skeleton:* skills | Our TDD overrides code-first |
| 4 | Superpowers (global plugin) | Generic brainstorm→plan→implement |
| 5 | Third-party skills | php-best-practices, vercel-react-best-practices |
| 6 | System default | Base Claude Code behavior |

## Checklist

1. Read `./CLAUDE.md` (orchestrator root)
2. Detect user intent:
   - **Backend:** mentions PHP, Laravel, API, endpoint, backend module, Action, DTO, migration, artisan
   - **Frontend:** mentions React, Next.js, component, page, hook, UI, form, table
   - **Cross-project:** mentions full feature, full-stack, API + UI, both repos
   - **Bug:** mentions error, failure, bug, fix, broken, not working, crash, exception
   - **Skill:** mentions create skill, edit skill, new workflow, modify workflow
3. Read `{repo}/CLAUDE.md` + `{repo}/ARCHITECTURE.md` for the target repo:
   - If backend: read `backend/CLAUDE.md` + `backend/ARCHITECTURE.md`
   - If frontend: read `frontend/CLAUDE.md` + `frontend/ARCHITECTURE.md`
   - If cross-project: read BOTH
   - If bug: detect affected repo from error context, read that one
4. Check scaffolding status:
   - Detect execution environment: `which php >/dev/null 2>&1` — if not found, use `./vendor/bin/sail` for all PHP commands
   - Backend deps check:
     - If PHP on host: `cd backend && composer show spatie/laravel-data 2>/dev/null`
     - If Sail only: `cd backend && ./vendor/bin/sail composer show spatie/laravel-data 2>/dev/null`
     - Fallback: `grep '"spatie/laravel-data"' backend/composer.json` (checks declared, not necessarily installed)
   - Frontend deps check: `ls frontend/node_modules/@tanstack/react-query 2>/dev/null`
   - Backend Pest check: `ls backend/tests/Pest.php 2>/dev/null` (Pest configured?)
   - If critical deps are missing, warn the user before proceeding
5. Check for active plans in `docs/plans/`
6. Route to the appropriate skill:

| Intent | Plan exists? | Destination skill |
|--------|-------------|-------------------|
| New feature | No | `skeleton:brainstorming` |
| New feature | Yes, incomplete | `skeleton:implementing` |
| Bug / error | N/A | `skeleton:debugging` |
| Create/edit skill | N/A | `skeleton:writing-skills` |

7. Inform the user what context was loaded and where we're routing

## Red Flags — Anti-Rationalizations

| Agent's excuse | Reality |
|----------------|---------|
| "I already know this project" | Fresh context window. ARCHITECTURE.md may have changed. Always read it. |
| "It's a trivial change, I don't need the architecture" | Trivial changes in wrong locations are bugs. Architecture defines file paths. |
| "The user told me exactly what to do" | Knowing WHAT isn't knowing WHERE or HOW. Without ARCHITECTURE.md you don't know conventions. |
| "I just need to edit one file I already know" | Without context, you edit the wrong file in the wrong module. 2 minutes of reading prevents 30 of fixing. |
| "I can read it later if I need it" | Later you've already made decisions without context. Read BEFORE is the rule. |
| "This is just a quick question, not a task" | Questions inform tasks. Context makes answers accurate. Load it. |
| "The user seems impatient, let me skip to the answer" | A wrong fast answer wastes more time than a correct slower one. Load context. |
| "I'll check the architecture as I go" | Checking as you go means discovering conflicts after you've written code. Front-load the reading. |
| "The user already specified exactly what to build" | Intent detection confirms the workflow path, not just the task. Even when the task is clear, routing determines which skills activate. Skip detection → skip skills → skip architecture checks. |

## Files Read

| File | When |
|------|------|
| `./CLAUDE.md` | Always |
| `backend/CLAUDE.md` | If target includes backend |
| `backend/ARCHITECTURE.md` | If target includes backend |
| `frontend/CLAUDE.md` | If target includes frontend |
| `frontend/ARCHITECTURE.md` | If target includes frontend |
| `docs/plans/*.md` | If any plan exists |

## Files Generated

None. Bootstrap only loads context and routes.

## Skill Discovery

The `skeleton:*` skills live in `skills/` (root), `backend/skills/`, and `frontend/skills/`. Symlinks in `.claude/skills/` are created by `setup.sh` but are gitignored.

**If skills are not discoverable via the Skill tool:** Read them directly from their file paths. Example: `Read skills/skeleton-brainstorming/SKILL.md` instead of invoking via Skill tool.

## Advancement Criteria

All must be true before transitioning:

- ARCHITECTURE.md of the target repo is loaded in the context window
- User intent classified into one of the 5 categories (backend, frontend, cross, bug, skill)
- Destination skill identified
- User informed of loaded context and routing decision

## Rollback Criteria

- Cannot determine target repo → ask the user
- ARCHITECTURE.md doesn't exist → warn user, proceed with CLAUDE.md only
- Critical dependencies missing → warn user before proceeding (do not block)

## Correct Example

```
User: "Necesito crear un módulo de Inventario"

Agent:
1. ✅ Reads ./CLAUDE.md
2. ✅ Detects: "nuevo módulo backend" (mentions módulo)
3. ✅ Reads backend/CLAUDE.md + backend/ARCHITECTURE.md
4. ✅ Checks: composer show spatie/laravel-data → installed
5. ✅ Checks docs/plans/ → no active plan
6. ✅ Routes to skeleton:brainstorming
7. ✅ Says: "Loaded backend architecture context. No active plan found.
   Routing to skeleton:brainstorming to explore requirements."
```

## Incorrect Example

```
User: "Necesito crear un módulo de Inventario"

Agent:
1. ❌ Skips reading ARCHITECTURE.md ("I know Laravel")
2. ❌ Starts creating files immediately in app/Models/
3. ❌ Uses SoftDeletes (laravel-specialist default)
4. ❌ Creates App\Http\Controllers\InventoryController instead of
   Modules\Inventory\Http\Controllers\
```

## Transitions

| Condition | Destination |
|-----------|-------------|
| New feature, no plan | `skeleton:brainstorming` |
| Existing incomplete plan | `skeleton:implementing` |
| Bug / error | `skeleton:debugging` |
| Create / edit skill | `skeleton:writing-skills` |
| Cannot determine target repo | Ask the user |
| ARCHITECTURE.md doesn't exist | Warn user, proceed with CLAUDE.md only |
