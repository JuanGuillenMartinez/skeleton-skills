---
name: skeleton-validating
description: >
  Use after completing implementation tasks or before claiming work is done.
  Runs architecture guards and final verification. Two modes: "task" (after
  each task) and "final" (after all tasks). Replaces architecture-guarding
  + verifying.
---

# Skeleton Validating

Runs architecture guards and verification. Use after every task and as final check before completion.

**Announce:** "Running skeleton:validating [task|final] for [backend|frontend]."

## Mode: Task (after each implementation task)

Run guards for the repo that was just modified:

### Backend Guards

| Command | Pass criteria |
|---------|---------------|
| `cd backend && ./vendor/bin/sail artisan test --filter={Test}` | PASS |
| `cd backend && ./vendor/bin/sail php ./vendor/bin/pint --test` | Exit 0 |

PHPStan and module dependency checks run only in **final** mode.

### Frontend Guards

| Command | Pass criteria |
|---------|---------------|
| `cd frontend && npm run test -- --run {test-file}` | PASS |
| `cd frontend && npm run lint` | Exit 0 |
| `cd frontend && npm run typecheck` | Exit 0 |

### Result Protocol

- **PASS** — All guards passed. Proceed to next task.
- **FAIL** — Report exact error. Fix before advancing. If same guard fails 3x, escalate to human.

## Mode: Final (after all tasks complete)

Full verification with evidence. Every assertion needs command output.

### Backend

| # | Command | Criteria |
|---|---------|----------|
| 1 | `cd backend && ./vendor/bin/sail artisan test` | ALL PASS |
| 2 | `cd backend && ./vendor/bin/sail php ./vendor/bin/pint --test` | Exit 0 |
| 3 | `cd backend && ./vendor/bin/sail php ./vendor/bin/phpstan analyse` | 0 errors |
| 4 | Import validation (no cross-module imports outside Contracts/Data) | No violations |

### Frontend

| # | Command | Criteria |
|---|---------|----------|
| 1 | `cd frontend && npm run test` | ALL PASS |
| 2 | `cd frontend && npm run lint` | Exit 0 |
| 3 | `cd frontend && npm run typecheck` | Exit 0 |
| 4 | `cd frontend && npm run build` | Exit 0 |

### Architecture Checklist (manual — final mode only)

After all commands pass, read the generated/modified code and verify:

**Backend:**
- [ ] Controllers thin: only validate → authorize → Action → response
- [ ] Actions invocable (`__invoke()`), all business logic
- [ ] DTOs pure: no `#[Rule()]`, no validation
- [ ] FormRequests: `rules()` via `match($this->method())`
- [ ] `Gate::authorize()` in controllers. NEVER `$this->authorize()`
- [ ] Models: `HasAuditUser` + `LogsActivity`, status enums, no SoftDeletes
- [ ] No destroy endpoints or DELETE routes
- [ ] Permissions: `{module_snake_case}.{action}`
- [ ] Inter-module: only `Contracts/` + `Data/` imports across modules

**Frontend:**
- [ ] No cross-module imports (except `types.ts`)
- [ ] No barrel exports (no `index.ts` re-exports)
- [ ] `app/` has no business logic
- [ ] Data fetching via `api.*` only (no direct `fetch`)
- [ ] Forms: Zod → RHF → shadcn
- [ ] Labels from `lib/labels.ts` (no hardcoded text)
- [ ] Env from `config/env.ts` (no `process.env`)

**Import validation (backend):**
Search for `use App\Modules\` across all modules. Cross-module imports are valid ONLY for:
- `Contracts\` — always allowed
- `Data\` — always allowed
- `Models\` — allowed only for FK relationships (`belongsTo`/`hasMany`)

Any other cross-module import (`Services/`, `Actions/`, `Http/`, etc.) = violation.

### OpenAPI → Frontend Type Sync (after backend final passes)

RULE: Any change to Controllers or FormRequests requires regenerating frontend types.
NEVER: Commit backend changes without syncing `types/api.ts`.

| # | Step | Criteria |
|---|------|----------|
| 1 | Verify Sail is running | `docker compose ps` shows containers up |
| 2 | `cd frontend && npm run generate:types` | Exit 0, `types/api.ts` updated |
| 3 | If `types/api.ts` changed: `cd frontend && npm run typecheck` | Exit 0 |
| 4 | If typecheck fails | Report errors. Do NOT continue to `skeleton:finishing` |

### Cross-project (full-stack)

- `cd frontend && npm run generate:types` — regenerate from OpenAPI
- No type errors after regenerating

### VCS Check

- `git diff --stat` per repo — changes match the plan
- No unexpected files (.env, node_modules/, vendor/)

### Report

- **ALL PASS:** "Verification PASS" with evidence (command outputs). Proceed to `skeleton:finishing`.
- **ANY FAIL:** Report exact failure. Do NOT claim completion.

## Anti-Rationalizations

| Temptation | Reality |
|------------|---------|
| "Tests passed earlier, no need to re-run" | Run again now. Fresh evidence or no evidence. |
| "I only changed one file, it can't break" | That's exactly when it breaks. Run the tests. |
| "PHPStan level 6 is too strict" | Level 6 is the project standard. Fix the type. |
| "The lint error is cosmetic" | Code style is convention. Fix it. 30 seconds. |

## Transitions

| Condition | Destination |
|-----------|-------------|
| Task mode PASS | Next task (laravel/nextjs implementing) |
| Final mode ALL PASS | `skeleton:finishing` |
| Guard FAIL | Back to implementing skill to fix |
| Same guard fails 3x | Escalate to human |
