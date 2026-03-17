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

1. **Read existing code** relevant to the area of change. Check for similar patterns.
2. **Ask the user** (consolidate when possible — propose defaults for speed):
   - What problem does this solve?
   - Who uses it? (roles, permissions)
   - What data does it handle? (fields, types, relationships)
   - Does it interact with other modules? Which ones?
3. **Inject architecture constraints** from ARCHITECTURE.md:
   - Backend: base module structure (12 files), extension points only when triggered, inter-module via Contracts+Data only
   - Frontend: base module structure (10 files), extension points only when triggered. Generate with `npx tsx scripts/make-module.ts --name={mod} --entity={Entity}`
4. **Propose design** (1 approach if architecture dictates, 2-3 if meaningful alternatives exist):
   - Files to create/modify (exact paths)
   - Extension points needed (if any, with trigger justification)
   - Required tests
5. **Design checklist** (verify before proceeding):
   - [ ] Files in correct locations per ARCHITECTURE.md?
   - [ ] No prohibited cross-module imports?
   - [ ] Naming follows conventions?
   - [ ] No SoftDeletes? (status enums instead)
   - [ ] ONE Data DTO, ONE FormRequest per entity (base case)?
   - [ ] Tests in `Modules/{Mod}/Tests/{Entity}Test.php` (backend) or `modules/{mod}/__tests__/{mod}.test.ts` (frontend)?
6. **User approval:** Present design, wait for explicit confirmation.

## Phase 2 — Plan

7. **Decompose into atomic tasks** (2-5 min each):

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

8. **Ordering rules:**
   - Backend tasks BEFORE frontend tasks (full-stack)
   - Dependency order (migration before model before action before controller)
   - Compact format OK when referencing Catalog patterns: "Follow Catalog pattern, adapt Entity/mod names"
9. **Save plan** to `docs/plans/YYYY-MM-DD-{feature-name}.md`
10. **Handoff:** "Plan saved. Ready to execute?"

## Plan Header

```markdown
# [Feature Name] Implementation Plan

**Goal:** [One sentence]
**Repos:** [backend | frontend | both]
**Architecture:** [2-3 sentences]
```

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
