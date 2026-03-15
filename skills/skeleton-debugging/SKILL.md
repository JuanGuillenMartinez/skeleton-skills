---
name: skeleton-debugging
description: >
  Use when encountering any bug, test failure, or unexpected behavior,
  before proposing fixes. Follows 4 systematic phases (reproduce, root
  cause, hypothesis, fix) with stack-specific tools for Laravel and Next.js.
---

# Skeleton Debugging

Systematic 4-phase debugging adapted to the project's stack. Never jump to fixes without reproducing first.

**Announce:** "Using skeleton:debugging to systematically investigate this issue."

## Phase 1: Reproduce

1. Get bug description (error message, stack trace, expected vs actual behavior)
2. Detect affected repo:
   - PHP error / HTTP 500 / artisan → backend
   - JS error / React / browser → frontend
   - API contract mismatch → investigate both
3. Read ARCHITECTURE.md for the affected repo
4. Reproduce the bug:
   - Backend: `cd backend && php artisan test --filter={RelevantTest}` or manual request
   - Frontend: `cd frontend && npm run test` or reproduce in browser
5. **Document:** Exact input, expected output, actual output, stack trace

## Phase 2: Root Cause

6. Trace the complete flow per architecture:

   **Backend flow:**
   Request → Route → Controller → FormRequest → Policy → Action → Service/Contract → Model → Response

   **Backend tools:**
   - `dd()` — dump and die at any point
   - `DB::listen(fn ($q) => logger($q->sql))` — log all queries
   - `php artisan tinker` — interactive REPL
   - `php artisan pail` — live tail logs
   - Telescope (if installed) — request inspector

   **Frontend flow:**
   User action → Component → Hook → api.ts → http-client → Response handling → State update → Re-render

   **Frontend tools:**
   - Browser console — errors and logs
   - React DevTools — component tree and state
   - TanStack Query devtools — query cache state
   - Network tab (Chrome DevTools MCP) — request/response inspection
   - `console.table(queryClient.getQueryCache().getAll().map(q => ({ key: q.queryKey, state: q.state.status })))` — dump all query states

7. Identify the exact layer where behavior diverges

## Phase 3: Hypothesis

8. Formulate specific hypothesis: "The bug occurs because [layer] does [X] when it should do [Y]"
9. Design a test that reproduces the bug (TDD RED):
   - Backend: Pest test that fails as expected
   - Frontend: Vitest test that fails as expected
10. Run test → **Verify: FAIL** as expected (confirms hypothesis)
    - If test passes → hypothesis is wrong. Go back to Phase 2.

## Phase 4: Fix

11. Implement minimal fix
12. Run test → **Verify: PASS** (GREEN)
13. Run full test suite → **Verify: ALL PASS** (no regressions)
14. Run architecture guards → **Verify: PASS**
15. Commit: `git commit -m "fix(mod): description of fix"`

## Escalation Rules

- 3+ fix attempts fail → question the hypothesis, re-analyze from Phase 2
- 5+ total attempts → the bug may be a design problem. Escalate to human with all evidence collected.

## Red Flags — Anti-Rationalizations

| Agent's excuse | Reality |
|----------------|---------|
| "I already know what it is, let me fix it directly" | Systematic: 15-30 min. Random fixes: 2-3 hours of thrashing. Reproduce first. |
| "I don't need to reproduce it, the error is clear" | If it's clear, reproducing takes 1 minute. If you can't reproduce it, you don't understand the bug. |
| "Let me just try something quick" | "Something quick" × 5 = an hour wasted without evidence. Reproduce and trace first. |
| "I can't write a test for this bug" | If you can't test the bug, you can't verify the fix. Find how to test it. |
| "It's a configuration bug, not code" | Configuration CAN be reproduced and tested. `.env`, `config/`, `phpunit.xml` are files like any other. |
| "The bug only happens in production" | Reproduce the environment: env vars, test data, config. If you can't reproduce locally, ask for logs. |
| "The stack trace points to the library, not our code" | Our code CALLS the library. The bug is in HOW we call it. Trace our call. |
| "This is probably a race condition" | Race conditions are reproducible with proper setup. Don't use "probably" — verify with evidence. |

## Correct Example

```
Bug: "Creating a sale doesn't decrement stock"

Phase 1: POST /api/sales → sale created, stock unchanged
Phase 2: Trace: Controller → CreateSaleAction → inventory->reserve() NOT CALLED
Phase 3: Hypothesis: "Action doesn't call reserve()"
         Test: it('decrements stock when sale is created') → FAIL ✓
Phase 4: Add $this->inventory->reserve() → PASS ✓
         Full suite → ALL PASS ✓
         Commit: "fix(sales): call inventory reserve on sale creation"
```

## Incorrect Example

```
Bug: "Creating a sale doesn't decrement stock"

1. ❌ "I bet it's the InventoryService" → changes InventoryService randomly
2. ❌ Still broken → changes the Controller
3. ❌ Still broken → adds dd() everywhere without reading the flow
4. ❌ 45 minutes later, still no fix, code is messy with debug statements
```

## Transitions

| Condition | Destination |
|-----------|-------------|
| Root cause identified, fix needs plan | `skeleton:planning` (for multi-file fix) |
| Simple fix (1-2 files) | Execute Phase 4 directly, then `skeleton:verifying` |
| Bug is unresolvable | Escalate to human with all evidence |
