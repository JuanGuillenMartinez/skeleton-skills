---
name: skeleton-architecture-guarding
description: >
  Use after completing any implementation task, before marking it done.
  Runs architecture validation tools appropriate to the target repository:
  Pint + PHPStan + module deps (backend) or ESLint + TypeScript (frontend).
---

# Skeleton Architecture Guarding

Validates that code changes respect architecture boundaries and quality standards. Blocks progress until all guards pass.

**Announce:** "Running skeleton:architecture-guarding for [backend|frontend]."

## Checklist

1. **Detect target repo** (backend or frontend) based on the last commit/change
2. **Execute guards for the repo:**

### Backend Guards

| # | Command | What it validates | Pass criteria |
|---|---------|------------------|---------------|
| 0 | `cd backend && ./vendor/bin/sail artisan test` | All tests pass | ALL PASS, 0 failures |
| 1 | `cd backend && ./vendor/bin/pint --test` | PSR-12 code style | Exit 0, no changes |
| 2 | `cd backend && ./vendor/bin/phpstan analyse` | Types, level 6 | 0 errors |
| 3 | `cd backend && php artisan modules:check-dependencies` | No cycles between modules | Exit 0 |
| 4 | Manual import validation | Only `Contracts/` (extension point) and `Data/` imported from other modules | No prohibited imports |

### Frontend Guards

| # | Command | What it validates | Pass criteria |
|---|---------|------------------|---------------|
| 0 | `cd frontend && npm run test` | All tests pass | ALL PASS, 0 failures |
| 1 | `cd frontend && npm run lint` | ESLint + Prettier | Exit 0 |
| 2 | `cd frontend && npm run typecheck` | TypeScript strict | Exit 0 |
| 3 | Manual import validation | Modules don't import from each other except via types.ts | No direct cross-module imports |
| 4 | Manual app/ validation | No business logic in app/ | Only routing, layouts, pages that delegate to modules/ |

3. **Report result:**
   - **PASS** — All guards executed and passed
   - **PARTIAL PASS** — Some guards passed, others not installed. List what ran and what was skipped. Document skipped guards as known gaps. Advance with caution.
   - **FAIL** — One or more guards failed. Report the exact error. DO NOT advance.

**PARTIAL PASS protocol:** When reporting PARTIAL PASS, include:
- ✅ Guards that ran and passed
- ⏭️ Guards skipped (tool not installed) — with install command
- Example: "PARTIAL PASS: pint ✅, phpstan ⏭️ (install: composer require --dev larastan/larastan), modules:check ⏭️ (not implemented)"

### Manual Import Validation Details

**Backend — check files changed in last commit:**
```bash
cd backend && git diff --name-only HEAD~1 | grep -v '/Contracts/' | grep -v '/Data/' | while read f; do
  grep -n 'use.*Modules\\' "$f" 2>/dev/null | grep -v "$(dirname "$f" | sed 's|/|\\\\|g')" | grep -v '\\Contracts\\' | grep -v '\\Data\\'
done
```
Any output = FAIL (importing non-Contract/Data from another module).

**Frontend — check files changed in last commit:**
```bash
cd frontend && git diff --name-only HEAD~1 | grep 'modules/' | while read f; do
  mod=$(echo "$f" | cut -d/ -f2)
  grep -n "from.*modules/" "$f" 2>/dev/null | grep -v "modules/$mod/" | grep -v "types"
done
```
Any output = FAIL (cross-module import not via types).

## Red Flags — Anti-Rationalizations

| Agent's excuse | Reality |
|----------------|---------|
| "PHPStan level 6 is too strict for this case" | Level 6 is the project standard. Fix the type, don't lower the level. |
| "It's a false positive in dependencies" | Investigate. If it's genuine, document why and find a workaround. Don't ignore. |
| "The lint error is just cosmetic" | Code style is team convention. Fix it. Takes 30 seconds. |
| "The typecheck fails on a generated type, not my code" | Then regenerate the types. If the generated type is wrong, the backend API changed. Synchronize. |
| "It only fails in CI, works locally" | Your environment is misconfigured. Fix your setup, not the guard. |
| "I can fix it later, let me move on" | Fixing later is tech debt. Guards exist to prevent it. Fix it NOW. |
| "The import validation is too strict" | The boundary rules are from ARCHITECTURE.md. They exist for a reason. Follow them. |
| "This guard isn't installed yet" | Then it's a blocker. Report it, don't skip it. The guard must run. |
| "Only Pint is installed, so running just Pint counts as full PASS" | Partial execution = PARTIAL PASS. Report what ran and what was skipped. Never claim full PASS with missing guards. |

## Correct Example

```
After completing a backend task:

1. ✅ pint --test → Exit 0
2. ✅ phpstan analyse → 0 errors
3. ✅ modules:check-dependencies → No cycles
4. ✅ Import validation → No prohibited imports
→ Report: "Architecture guard PASS: pint, phpstan, modules:check, imports"
```

## Incorrect Example

```
After completing a backend task:

1. ✅ pint --test → Exit 0
2. ❌ phpstan analyse → 2 errors (undefined method on DTO)
3. Agent says: "These are minor, I'll fix them later"
4. ❌ Proceeds to next task

Problem: PHPStan errors accumulate. By task 5, there are 12 errors
and the root cause is buried in task 2.
```

## Transitions

| Condition | Destination |
|-----------|-------------|
| All guards PASS | `skeleton:implementing` (next task) or `skeleton:verifying` (if last task) |
| PARTIAL PASS | `skeleton:implementing` (next task) with documented gaps |
| Guard FAIL | Back to `skeleton:laravel-implementing` or `skeleton:nextjs-implementing` to fix |
| Guard command not found | Report as blocker, do not skip |
| Same guard fails 3+ times | Escalate to human (possible design problem) |
