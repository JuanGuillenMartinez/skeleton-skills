---
name: skeleton-brainstorming
description: >
  Use before any creative work — creating features, building components,
  adding functionality, or modifying behavior. Explores user intent,
  requirements, and design constraints from ARCHITECTURE.md before
  implementation. Must run after skeleton:bootstrapping.
---

# Skeleton Brainstorming

Explores requirements through Socratic dialogue, injects architecture constraints, and produces a design that respects the modular architecture before any code is written.

**Announce:** "Using skeleton:brainstorming to explore requirements and design constraints."

**Prerequisite:** `skeleton:bootstrapping` must have run (ARCHITECTURE.md loaded in context).

## Checklist

1. **Verify prerequisite:** ARCHITECTURE.md for the target repo is in context. If not, run `skeleton:bootstrapping` first.
2. **Explore context:**
   - Read existing files relevant to the area of change
   - Identify affected modules (backend and/or frontend)
   - Check for similar patterns in existing code
3. **Ask the user questions** (one at a time, or consolidated with proposed defaults if user signals preference for speed):
   - What problem does this feature solve?
   - Who uses it? (roles, permissions)
   - Does it interact with other modules? Which ones?
   - Does it need data scoping? (multi-store, multi-branch) — only if project uses scoping
   - What data does it handle? (entities, relationships)
4. **Inject architecture constraints** based on target repo:

   **Backend constraints (from ARCHITECTURE.md):**
   - Module structure: `Modules/{Mod}/` with Actions, Contracts, Services, DTOs, Models, Http, Tests, Routes, Config, Lang
   - Inter-module communication: ONLY via Contracts (interfaces) + DTOs. NEVER import Models/Services/Actions from other modules.
   - Actions: invocable classes (`__invoke()`), all business logic here
   - DTOs: spatie/laravel-data, separated by purpose — `Create*Data` / `Update*Data` (input), `*Data` (output)
   - No SoftDeletes — use status with enums
   - Validation: ONLY in Form Requests
   - Permissions: `{mod}.{action}` via spatie/laravel-permission
   - i18n: `__('mod::messages.key')`, never hardcoded text
   - Testing: Pest, Feature tests MANDATORY for endpoints, Unit only for complex Actions

   **Frontend constraints (from ARCHITECTURE.md):**
   - Module structure: `modules/{mod}/` with api.ts, hooks/, components/, types.ts
   - Data fetching: `api.*` from http-client, NEVER direct fetch()
   - State: TanStack Query (server), Zustand (global UI), useState (local UI)
   - Query key factories: `{mod}Keys.all`, `{mod}Keys.list(params)`, `{mod}Keys.detail(id)`
   - Hooks: `use{Mod}s`, `use{Action}{Mod}` — behavior in hooks, UI in components
   - Permissions: `can("{mod}.{action}")` via array from /api/user
   - Forms: RHF + Zod + mapApiErrors
   - Labels: from `lib/labels.ts`, never hardcoded strings
   - Testing: Vitest + Testing Library + MSW, never `vi.mock`

5. **Propose approaches** with pros/cons, evaluated against the constraints:
   - 2-3 approaches when meaningful alternatives exist
   - 1 approach with explicit justification from ARCHITECTURE.md when the architecture leaves no meaningful design alternatives (e.g., the modular structure dictates file locations, DTOs, Actions pattern)
   - Approaches should differ in scope/strategy, not just in minor implementation details
6. **Present design** in sections:
   - Files to create/modify (exact paths)
   - Interfaces/Contracts (if applicable)
   - Required tests
   - Migrations (if backend)
   - UI components (if frontend)
7. **Spec review loop** (up to 5 iterations):
   - Dispatch spec-reviewer subagent (see `spec-reviewer-prompt.md`)
   - If issues: correct and re-review
   - If approved: continue
8. **User approval:** Present final design, wait for explicit confirmation
9. **Transition** to `skeleton:planning`

## Advancement Criteria

All must be true before transitioning to `skeleton:planning`:

- User approved the design explicitly
- Architecture constraints verified against the design (no violations)
- Files to create/modify identified with exact paths
- Spec review loop completed (approved or max 5 iterations with escalation)

## Rollback Criteria

- Design violates irreconcilable architecture constraints → escalate to human
- User rejects all 3 proposed approaches → re-explore intent from scratch
- Spec review loop reaches 5 iterations without approval → escalate to human

## Common Pitfalls

| Temptation | Reality |
|------------|---------|
| "ARCHITECTURE.md already defines everything, brainstorming is redundant" | Architecture defines patterns, not requirements. Brainstorming caught missing LogsActivity, missing query-builder dep, incomplete Config/module.php structure. Without it, you build incomplete modules. |
| "The design is obvious, I'll skip to planning" | "Obvious" designs have hidden assumptions. Brainstorming surfaced 8 issues in the Catalog module that the "obvious" design missed. |
| "Questions slow the user down" | Wrong implementations slow the user down more. 5 minutes of questions prevent 2 hours of rework. Consolidate if user wants speed, but don't skip. |
| "There's only one way to do this" | If there's truly one way, documenting WHY takes 30 seconds and prevents future questioning. Always justify, even if it's "per ARCHITECTURE.md section X". |

## Correct Example

```
User: "Necesito un módulo de Clientes para el backend"

Agent:
1. ✅ Verifies ARCHITECTURE.md is in context
2. ✅ Checks for existing customer-related code
3. ✅ Asks: "What fields does a Customer have? Name, email, phone, status?"
4. ✅ Asks: "What permissions? customers.view, customers.create, customers.update?"
5. ✅ Asks: "Does Customer interact with other modules (Sales, Invoicing)?"
6. ✅ Injects constraints: "Per ARCHITECTURE.md, Customer will expose
   a CustomerServiceContract with DTOs. Sales module will only import
   the contract + DTOs, never the Customer model directly."
7. ✅ Proposes design:
   - Modules/Customer/Contracts/CustomerServiceContract.php
   - Modules/Customer/DTOs/CreateCustomerData.php
   - Modules/Customer/DTOs/CustomerData.php
   - Modules/Customer/Actions/CreateCustomerAction.php
   - Modules/Customer/Http/Controllers/CustomerController.php
   - Modules/Customer/Tests/Feature/Http/CustomerControllerTest.php
8. ✅ Runs spec review
9. ✅ Waits for user approval
```

## Incorrect Example

```
User: "Necesito un módulo de Clientes para el backend"

Agent:
1. ❌ Skips questions, assumes requirements
2. ❌ Designs with App\Models\Customer (wrong structure)
3. ❌ Uses API Resources instead of DTOs
4. ❌ Adds SoftDeletes to migration
5. ❌ Starts coding immediately without user approval
```

## Files Read

- ARCHITECTURE.md of target repo (already loaded by bootstrap)
- Existing files in the most similar module (as pattern reference)
- `config/modules.php` (backend) if there are inter-module dependencies
- Existing types/interfaces if integrating with existing modules

## Files Generated

- `docs/specs/{feature-name}.md` — Only if the feature touches multiple files in both repos. For simple single-repo features, the design is documented directly in the plan.

## Transitions

| Condition | Destination |
|-----------|-------------|
| Design approved by user | `skeleton:planning` |
| Trivial feature (1 file, 1 repo) | `skeleton:planning` (skip spec file) |
| User rejects all 3 approaches | Re-explore intent from scratch |
| Spec review loop hits 5 iterations | Escalate to human |

---
