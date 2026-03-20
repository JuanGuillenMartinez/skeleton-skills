---
name: skeleton-designing
description: >
  Use before any new feature or module. Explores requirements, injects
  architecture constraints, proposes design, and creates an atomic TDD
  implementation plan — all in one pass. Replaces brainstorming + planning.
---

# Skeleton Designing

Explores requirements and produces a ready-to-execute TDD plan in a single cycle. No separate brainstorming/planning phases.

**Announce:** "Using skeleton:designing to explore requirements and create the implementation plan."

**Prerequisite:** `skeleton:bootstrapping` must have run (ARCHITECTURE.md loaded).

## Phase 1 — Explore & Design

0. **Design Discovery (MANDATORY for frontend tasks):**
   1. Check if `/design` folder exists in the project root
      - If YES: list files, read image descriptions/PDFs/wireframes
      - Present: "Found design references in /design: [list]. Using as visual foundation."
   2. Check if user provided design files/links (Figma, screenshots, mockups)
      - If YES: acknowledge and reference them
   3. Check if `frontend/DESIGN_SYSTEM.md` exists
      - If YES: read it. All visual decisions flow from this file
      - If NO + /design exists: suggest `design-system-extractor` first
      - If NO + no /design: note we need visual direction during brainstorming
   4. If implementing a NEW page with no design reference:
      - Read existing pages/components for visual consistency
      - Generate text-based layout description during brainstorming
      - Get user approval on layout BEFORE any code
   5. Check if `anthropics/frontend-design` plugin is installed: `grep -r "frontend-design" .claude/settings.json .mcp.json 2>/dev/null`
      - If YES: note in spec. During implementation, aesthetic decisions combine DESIGN_SYSTEM.md tokens + plugin guidance
      - When brainstorming visual direction, reference plugin archetypes: Brutally Minimal, Maximalist, Retro-Futuristic, Luxury/Refined, Editorial/Magazine — ask user which fits
   - RULE: NEVER start UI without (a) design reference from /design or user, (b) approved layout from brainstorming, or (c) DESIGN_SYSTEM.md with enough patterns. If none exist, STOP.
1. **Read existing code** relevant to the area of change. Check for similar patterns.
2. **Ask the user** (consolidate when possible — propose defaults for speed):
   - What problem does this solve?
   - Who uses it? (roles, permissions)
   - What data does it handle? (fields, types, relationships)
   - Does it interact with other modules? Which ones?
   - **Deletion strategy:** Does this module require record deletion?
     → **No (default):** status enum (active/inactive) + no destroy endpoint
     → **Yes:** destroy endpoint — only if the module is standalone, with no critical history (reports, audits, FK references from other modules)
     → If other modules have FK references to this one → force No, explain why
3. **Inject architecture constraints** from ARCHITECTURE.md:
   - Backend: base module structure per ARCHITECTURE.md, scaffolded by `./vendor/bin/sail artisan make:module {Name} --entity={Entity}`. Extension points only when triggered. Inter-module via Contracts+Data only
   - Frontend: base module structure per ARCHITECTURE.md, scaffolded by `npx tsx scripts/make-module.ts --name={mod} --entity={Entity}`. Extension points only when triggered
   - **Frontend visual design** (when plan includes UI tasks):
     - Read `frontend/DESIGN_SYSTEM.md` from disk before designing
     - List which existing design system components will be reused
     - List which NEW components need creation (these need `.stories.tsx`)
     - Specify module structure: files in `modules/{mod}/` and pages in `app/`
     - NEVER: vague UI descriptions ("create a nice dashboard"). Specify exact components, layout pattern, and module location
4. **Propose design** (1 approach if architecture dictates, 2-3 if meaningful alternatives exist):
   - For new modules: first task MUST be running the generator. NEVER create module files manually
   - Files to create/modify (exact paths)
   - Extension points needed (if any, with trigger justification)
   - Required tests
5. **Design checklist** (verify before proceeding):
   - [ ] New module? First task = run generator
   - [ ] Files in correct locations per ARCHITECTURE.md?
   - [ ] Naming follows conventions?
   - [ ] No prohibited cross-module imports? (only Contracts/ + Data/)
   - [ ] No SoftDeletes? (status enums instead)
   - [ ] Deletion strategy confirmed? (No → status transitions, Yes → destroy only if standalone + no FK refs)
   - [ ] Four DTOs planned: InputData (create), UpdateData (update), Data (output), FilterData (index)?
   - [ ] ListAction planned for index query logic — not in Controller?
   - [ ] Resource planned for output transformation?
   - [ ] ONE FormRequest per entity (base case)?
   - [ ] Models have HasAuditUser + LogsActivity?
   - [ ] Permissions named `{module_snake_case}.{action}`?
   - [ ] Controllers thin? (validate → authorize → Action → response)
   - [ ] Tests in correct location per ARCHITECTURE.md?
   - [ ] [FE] No barrel exports? No cross-module imports?
   - [ ] [FE] Forms: Zod → RHF → shadcn? Labels from `lib/labels.ts`? Env from `config/env.ts`?
   - [ ] [FE] IDs and timestamps are `string`, never `number`?
   - [ ] [FE] TypeScript strict, no `any`?
   - [ ] [FE] State: server=TanStack Query, global UI=Zustand, local=useState?
   - [ ] [FE] Permissions: components using can()/Authorized where required?
   - [ ] [FE] Test strategy defined (which hooks/components are tested)?
   - [ ] [FE] Error handling: mapApiErrors covers per-field 422, business errors, and unexpected errors?
   - [ ] [FE] DESIGN_SYSTEM.md read? Visual decisions reference its tokens?
   - [ ] [FE] New shared components identified? Storybook stories planned?
6. **User approval:** Present design, wait for explicit confirmation.
7. **Save design spec** to `docs/specs/YYYY-MM-DD-{feature-name}.spec.md`:
   - Scope, visual references used, layout decisions, extension points, out-of-scope
   - Commit spec before proceeding to plan
   - RULE: The plan references the spec file. Design lives on disk, not in conversation context.

## Phase 2 — Plan

8. **Decompose into atomic tasks** (2-5 min each):

   ### Task N: [Component Name]
   **Files:** Create: `path` | Modify: `path` | Test: `path`
   - [ ] RED: Write failing test — [code or pattern reference]
   - [ ] GREEN: Minimal implementation
   - [ ] GUARD: pint/phpstan (backend) or lint/typecheck (frontend)
   - [ ] COMMIT: `feat(mod): description`

   For infrastructure tasks (deps, config, seeders — no TDD):
   - [ ] Create/install
   - [ ] Verify
   - [ ] GUARD + COMMIT

9. **Ordering rules:**
   - Backend tasks BEFORE frontend tasks (full-stack)
   - Dependency order (migration before model before action before controller)
   - Compact format OK when referencing stub patterns from make:module: "Follow stub pattern, adapt Entity/mod names"
10. **Save plan** to `docs/plans/YYYY-MM-DD-{feature-name}.md`
    - RULE: The plan file on disk is the SINGLE SOURCE OF TRUTH for progress
    - RULE: Each `- [ ]` becomes `- [x]` when completed. Implementing skills update the file after each step
    - RULE: Resuming work = read plan from disk, find first unchecked `- [ ]`, continue from there
    - NEVER: Rewrite the plan. Only toggle checkboxes and append notes
11. **Handoff:** "Plan saved. Ready to execute?"

## Plan Header

```markdown
# [Feature Name] Implementation Plan

**Goal:** [One sentence]
**Repos:** [backend | frontend | both]
**Architecture:** [2-3 sentences]
```

## Preview Decision

When creating the plan for a task with frontend UI:
- Add a "Preview" task BEFORE the frontend implementation tasks
- The preview task invokes `skeleton:previewing`
- Implementation tasks reference the approved prototype

Example plan structure for a feature with UI:
  Task 1: [Backend work]
  Task 2: [Preview] Generate and approve HTML prototype ← `skeleton:previewing`
  Task 3: [Frontend] Implement approved prototype in React ← `skeleton:nextjs-implementing`
  Task 4: [Frontend] Fidelity check against prototype

## Anti-Rationalizations

| Temptation | Reality |
|------------|---------|
| "The design is obvious, skip to coding" | "Obvious" designs miss edge cases. 5 min of design prevents hours of rework. |
| "I already know the file paths" | Wrong paths are the #1 architecture violation. Verify against ARCHITECTURE.md. |
| "Tasks are too granular" | One step = one action. Granularity prevents cascading errors. |
| "I can combine RED+GREEN into one step" | No. The RED phase catches wrong assumptions. Never skip it. |

## Transitions

| Condition | Destination |
|-----------|-------------|
| Plan approved | `skeleton:laravel-implementing` or `skeleton:nextjs-implementing` (per task) |
| Design rejected | Re-explore from step 2 |
| Plan exceeds 20 tasks | Suggest splitting into sub-projects |
