---
name: skeleton-verifying
description: >
  Use when about to claim work is complete, before committing final
  changes or creating PRs. Runs ALL verification commands and confirms
  output matches expectations. Evidence before assertions, always.
---

# Skeleton Verifying

Final verification before claiming completion. Every assertion must have evidence (command output).

**Announce:** "Using skeleton:verifying to run final verification."

**Rule:** Never claim work is done without running this skill. Evidence before assertions.

## Checklist

1. **Re-read the plan** (or bug description if debugging)
2. **Verify completeness:** For each task/item in the plan:
   - Does the file exist at the specified path?
   - Does the test exist and cover the described behavior?
   - Is the checkbox marked `[x]`? (If checkboxes weren't updated during execution, verify by checking that the files and tests exist instead — don't block on checkbox state alone)
3. **Backend verification** (if applicable):

   > **Note:** If PHP is not on host, prefix commands with `./vendor/bin/sail`. See skeleton:laravel-implementing for the full command mapping.

   | # | Command | Criteria |
   |---|---------|----------|
   | 1 | `cd backend && ./vendor/bin/sail artisan test` | ALL PASS |
   | 2 | `cd backend && ./vendor/bin/sail php ./vendor/bin/pint --test` | Exit 0 |
   | 3 | `cd backend && ./vendor/bin/sail php ./vendor/bin/phpstan analyse` | 0 errors (or SKIPPED if not installed) |
   | 4 | `cd backend && ./vendor/bin/sail artisan modules:check-dependencies` | Exit 0 (or SKIPPED if not implemented) |

4. **Frontend verification** (if applicable):

   | # | Command | Criteria |
   |---|---------|----------|
   | 1 | `cd frontend && npm run test` | ALL PASS |
   | 2 | `cd frontend && npm run lint` | Exit 0 |
   | 3 | `cd frontend && npm run typecheck` | Exit 0 |
   | 4 | `cd frontend && npm run build` | Exit 0, no warnings |

5. **Cross-project verification** (if full-stack):
   - OpenAPI spec updated: `cd backend && php artisan scramble:export` or equivalent
   - Frontend types regenerated: `cd frontend && npm run generate:types`
   - No type errors after regenerating
6. **VCS verification** — do not trust what the agent says it did:
   - `cd backend && git diff --stat` — review that changes are as expected
   - `cd frontend && git diff --stat` — same
   - No unexpected files (`.env`, `node_modules/`, `vendor/`)
7. **If ALL pass:** Report "Verification PASS" with evidence (output of each command)
8. **If ANY fails:** Report exact FAIL, do NOT claim completion

## Red Flags — Anti-Rationalizations

| Agent's excuse | Reality |
|----------------|---------|
| "Tests passed when I ran them earlier" | Run again. Now. Fresh evidence or no evidence. |
| "I only changed one file, it can't break anything" | "Can't break" is exactly when it breaks. Run the tests. Always. |
| "The build is slow, I know it works" | You don't know. Run it. 2 minutes of build vs hours of debugging in production. |
| "I already verified manually in the browser" | Manual isn't reproducible evidence. Commands with output. |
| "The git diff is large, I don't need to review it" | Review it. Unexpected files are bugs waiting to explode. |
| "Everything passed in the individual tasks, no need to re-run" | Interactions between tasks can create bugs. Full suite, always. |
| "The build warning is harmless" | Warnings become errors in the next framework version. Fix now. |
| "I'm confident it works" | Confidence is not evidence. Run the commands. |

## Correct Example

```
Verification for a backend module:

1. ✅ Re-read plan: 5 tasks, all [x] checked
2. ✅ php artisan test → 23 tests, 23 passed
3. ✅ pint --test → Exit 0
4. ✅ phpstan analyse → 0 errors, level 6
5. ✅ modules:check-dependencies → No circular dependencies
6. ✅ git diff --stat → 12 files changed (matches plan)
→ "Verification PASS: all 4 checks green, git diff matches plan"
```

## Incorrect Example

```
Agent: "I've completed all tasks. The module is ready."

❌ No commands were run
❌ No output shown as evidence
❌ "Completed" is an assertion without evidence
```

## Transitions

| Condition | Destination |
|-----------|-------------|
| Verification PASS complete | `skeleton:finishing` |
| Tests fail | `skeleton:implementing` (fix) |
| Guards fail | `skeleton:architecture-guarding` (investigate) |
| Unexpected git diff | Investigate before continuing |
