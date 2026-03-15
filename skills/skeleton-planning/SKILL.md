---
name: skeleton-planning
description: >
  Use after brainstorming approval, before implementation. Creates atomic
  TDD task plans with architecture guard checkpoints for both backend
  (Pest, Pint, PHPStan) and frontend (Vitest, ESLint, TypeScript).
---

# Skeleton Planning

Creates bite-sized TDD task plans with exact file paths, code, commands, and architecture guard checkpoints.

**Announce:** "Using skeleton:planning to create the implementation plan."

**Prerequisite:** Design approved in `skeleton:brainstorming` (or spec exists in `docs/specs/`).

## Plan Header Template

Every plan MUST start with:

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED: Use skeleton:implementing to execute this plan.

**Goal:** [One sentence]
**Repos:** [backend | frontend | backend + frontend]
**Architecture:** [2-3 sentences about approach]
**Tech Stack:** [Key packages/libraries]
```

## Checklist

1. **Read spec/design** approved during brainstorming
2. **Scope check:** If design covers multiple independent subsystems, suggest separate plans
3. **Map files:**
   - Complete list of files to create/modify with exact paths
   - One clear responsibility per file
   - Dependency order (what gets built first)
4. **Verify file paths against ARCHITECTURE.md:**
   - Backend tests: `app/Modules/{Mod}/Tests/Feature/` or `app/Modules/{Mod}/Tests/Unit/` — NEVER `tests/Feature/`
   - Backend code: `app/Modules/{Mod}/` — NEVER `app/Models/`, `app/Http/Controllers/`
   - Frontend tests: `modules/{mod}/__tests__/` — NEVER `__tests__/` at root
   - Frontend code: `modules/{mod}/` — NEVER root-level files for module logic
   - Cross-check every file path in the plan against the module structure template in ARCHITECTURE.md
5. **Decompose into atomic tasks** (2-5 min each):
   - Each task = 1 component/file/functionality
   - Each task follows TDD steps (see format below)
6. **Task format:**

   ### Task N: [Component Name]

   **Files:**
   - Create: `exact/path/to/file.ext`
   - Modify: `exact/path/to/existing.ext:lines`
   - Test: `exact/path/to/test.ext`

   - [ ] Step 1: Write failing test — [exact test code]
   - [ ] Step 2: Run test, verify FAIL — [exact command + expected error]
   - [ ] Step 3: Write minimal implementation — [exact code]
   - [ ] Step 4: Run test, verify PASS — [exact command]
   - [ ] Step 5: Run architecture guard — [pint/phpstan/lint/typecheck per repo]
   - [ ] Step 6: Commit — [exact git commands]

7. **Infrastructure Task format** (for non-TDD tasks like installing deps, creating config):

   ### Task N: [Infrastructure Component]

   **Files:**
   - Create: `exact/path/to/file.ext`
   - Modify: `exact/path/to/existing.ext`

   - [ ] Step 1: Create/install — [exact code or command]
   - [ ] Step 2: Verify — [exact verification command + expected output]
   - [ ] Step 3: Guard — [pint/lint]
   - [ ] Step 4: Commit — [exact git commands]

   Use this format for: dependency installation, shared infrastructure, config files, seeders, non-testable scaffolding. TDD format is still required for all code with testable behavior.

8. **For full-stack:** Backend tasks BEFORE frontend tasks. Frontend tasks reference the API created in backend.
9. **Architecture guard checkpoints** after each task:
   - Backend: `cd backend && ./vendor/bin/pint --test && ./vendor/bin/phpstan analyse && php artisan modules:check-dependencies`
   - Frontend: `cd frontend && npm run lint && npm run typecheck`
10. **Save plan** to `docs/plans/YYYY-MM-DD-{feature-name}.md`
11. **Plan review loop** (up to 5 iterations): Dispatch reviewer, fix issues, repeat until approved. If 5 iterations reached without approval → escalate to human.
12. **Handoff:** "Plan saved to `docs/plans/...`. Ready to execute?"

## Red Flags — Anti-Rationalizations

| Agent's excuse | Reality |
|----------------|---------|
| "This task is too small to plan" | The plan prevents files in wrong locations. 2 min of planning saves 20 of debugging. |
| "I already know what code to write, let me start" | Knowing isn't documenting. The plan is for verification, not for you. |
| "The plan is implicit in the spec" | Spec says WHAT. Plan says HOW: exact paths, commands, expected output. |
| "I don't need guards in every task" | Guards are checkpoints. Skipping them accumulates tech debt that explodes at the end. |
| "I can combine these 3 steps into one" | No. One step = one action. Granularity prevents errors. |
| "The commit can go at the end of all tasks" | No. Frequent commits = granular rollback. One commit per task. |
| "I'll figure out the file paths as I go" | Wrong paths are the #1 source of architecture violations. Decide paths upfront. |
| "The test patterns are obvious, I'll skip writing them" | Obvious tests take 30 seconds to write. Missing tests take hours to add later. |
| "Infrastructure tasks don't need planning" | Even non-TDD tasks need exact file paths and commands. Wrong paths in infra tasks cascade into every subsequent task. Plan them. |

## Correct Example

```
Input: Design for a Customer module backend (from brainstorming)

Plan output:
Task 1: Migration + Model
  - Create: Modules/Customer/Database/Migrations/2026_03_14_create_customers_table.php
  - Create: Modules/Customer/Models/Customer.php
  - Test: Modules/Customer/Tests/Feature/Http/CustomerControllerTest.php
  - Step 1: Write test that POSTs to /api/customers → FAIL (table doesn't exist)
  - Step 2: Run test → FAIL ✓
  - Step 3: Create migration + model
  - Step 4: Run test → still FAIL (no route) — that's OK, this task only creates table
  - Step 5: pint + phpstan
  - Step 6: Commit

Task 2: DTOs
  (CreateCustomerData, UpdateCustomerData, CustomerData)
  ...
```

## Incorrect Example

```
Plan output:
Task 1: Create entire Customer module
  - Create ALL files (migration, model, DTOs, action, controller, tests)
  - Run all tests at end
  - Single commit

Problems:
  ❌ Task too large (not 2-5 min)
  ❌ No TDD steps (no RED phase)
  ❌ No architecture guards between steps
  ❌ Single commit (no granular rollback)
```

## Compact Plan Format

When a reference module already exists (e.g., `Modules/Catalog/`), subsequent module plans can use a compact format that references the existing module's patterns instead of repeating full code:

```
- [ ] Step 3: Create Action following Catalog pattern
  See: `app/Modules/Catalog/Actions/CreateProductAction.php`
  Adapt: Replace `Product` with `{Entity}`, `catalog` with `{mod}`
```

This reduces plan size from ~500 lines to ~200 lines for well-understood CRUD modules. Only use when: (1) reference module exists, (2) the new module follows the same patterns, (3) no novel architectural decisions needed.

## Files Read

- Spec/design from brainstorming or `docs/specs/`
- ARCHITECTURE.md of target repo (for file paths and conventions)
- Existing files to be modified
- Example module if it exists (`Modules/Catalog/` in backend)

## Files Generated

- `docs/plans/YYYY-MM-DD-{feature-name}.md`

## Transitions

| Condition | Destination |
|-----------|-------------|
| Plan saved and approved | `skeleton:implementing` |
| Plan needs redesign | `skeleton:brainstorming` (go back) |
| Plan exceeds 20 tasks | Suggest splitting into sub-projects |
| Review loop hits 5 iterations | Escalate to human |

---
