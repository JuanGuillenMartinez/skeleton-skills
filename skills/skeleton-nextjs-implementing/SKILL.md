---
name: skeleton-nextjs-implementing
description: >
  Use when implementing a frontend task from a plan. Enforces TDD with
  Vitest, Testing Library, and MSW. RED→GREEN→REFACTOR→GUARD→COMMIT.
---

# Skeleton Next.js Implementing

TDD implementation for frontend tasks. Every task follows RED→GREEN→REFACTOR→GUARD→COMMIT.

**Announce:** "Using skeleton:nextjs-implementing for this frontend task."

**Prerequisite:** Task definition from plan (files, code, expected output).

## Checklist

1. **Read the task** from the plan
2. **RED — Write failing test:**
   - Location: `modules/{mod}/__tests__/{mod}.test.ts` (single file, `describe()` blocks)
   - Framework: Vitest + Testing Library + MSW (NEVER `vi.mock`)
   - Run: `cd frontend && npm run test -- --run {test-file}`
   - **Verify: FAIL**
3. **GREEN — Minimal implementation:**
   - Data fetching: `api.*` from `lib/http-client.ts` (never direct `fetch`)
   - Query keys: `{mod}Keys.all`, `{mod}Keys.list(params)`, `{mod}Keys.detail(id)`
   - Hooks: `use{Mod}s`, `use{Action}{Mod}` — behavior in hooks, UI in components
   - Labels: from `lib/labels.ts` (never hardcoded)
   - TypeScript strict, never `any`
   - Run: `cd frontend && npm run test -- --run {test-file}`
   - **Verify: PASS**
4. **REFACTOR — Clean if needed.** Run full suite: `npm run test` — ALL PASS
5. **GUARD:** `npm run lint && npm run typecheck`
6. **COMMIT:** `cd frontend && git add [files] && git commit -m "feat(mod): description"`
7. **Next:** Run `skeleton:validating task` then proceed to next task

## Test Pattern (Hook with MSW — mandatory)

```typescript
import { renderHook, waitFor } from '@testing-library/react';
import { http, HttpResponse } from 'msw';
import { server } from '@/test/setup';
import { useCustomers } from '../hooks';
import { createWrapper } from '@/test/providers';

it('fetches customers list', async () => {
  server.use(
    http.get('*/api/customers', () =>
      HttpResponse.json({
        data: [{ id: '1', name: 'Test' }],
        meta: { current_page: 1, per_page: 15, total: 1 },
      })
    )
  );

  const { result } = renderHook(() => useCustomers(), {
    wrapper: createWrapper(),
  });

  await waitFor(() => expect(result.current.isSuccess).toBe(true));
  expect(result.current.data?.data).toHaveLength(1);
});
```

## Anti-Rationalizations

| Temptation | Reality |
|------------|---------|
| "I'll write the test after" | No. Test FIRST. Always. |
| "MSW is too much setup" | MSW is configured once in `test/setup.ts`. Each test only adds handlers. |
| "It's pure UI, no logic to test" | If it has hooks, data, or permissions: TEST. Pure shadcn with no custom logic: skip. |

## Transitions

| Condition | Destination |
|-----------|-------------|
| Task committed | `skeleton:validating task` → next task |
| RED phase passes (shouldn't) | Rewrite the test |
| GREEN requires >100 lines | Task too large — decompose |
| All tasks done | `skeleton:validating final` |
