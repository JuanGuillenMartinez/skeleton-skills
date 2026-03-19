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

0. **Load design context (MANDATORY — NEVER SKIP):**
   - Read `frontend/ARCHITECTURE.md` from disk (even if read earlier — files change mid-session)
   - Read `frontend/DESIGN_SYSTEM.md` from disk (if it exists)
   - Read `frontend/CLAUDE.md` from disk
   - RULE: If DESIGN_SYSTEM.md exists, ALL visual decisions (colors, spacing, typography, variants) come from it. NEVER invent styles.
   - RULE: If DESIGN_SYSTEM.md does NOT exist, STOP. Tell user: "No DESIGN_SYSTEM.md found. Create one before implementing UI."
1. **Read the task** from the plan
2. **RED — Write failing test:**
   - Location: `modules/{mod}/__tests__/{mod}.test.ts` (single file, `describe()` blocks)
   - Framework: Vitest + Testing Library + MSW (NEVER `vi.mock`)
   - Run: `cd frontend && npm run test -- --run {test-file}`
   - **Verify: FAIL**
3. **GREEN — Minimal implementation:**
   Implement per ARCHITECTURE.md. Quick-ref:
   - Data fetching: `api.*` from `lib/http-client.ts`. NEVER direct `fetch()`
   - Query keys: `{mod}Keys.all`, `{mod}Keys.list(params)`, `{mod}Keys.detail(id)`
   - Hooks: `use{Mod}s`, `use{Action}{Mod}` — behavior in hooks, UI in components
   - Forms: Zod schema → RHF `useForm` → shadcn `FormField` → `mutateAsync` + `mapApiErrors`
   - Error handling: `mapApiErrors(error, form.setError)` mandatory in forms.
     Covers: per-field 422, business errors (toast), unexpected errors (generic toast)
   - IDs and timestamps: always `string`. NEVER `number` for IDs
   - Labels: `lib/labels.ts`. NEVER hardcoded user-facing text
   - Env: `import { env } from "@/config/env"`. NEVER `process.env`
   - Styling: Tailwind + `cn()` only. NEVER CSS modules or styled-components
   - No barrel exports. NEVER `index.ts` re-exports
   - No cross-module imports (except `types.ts`)
   - `app/` has no business logic — only routing and layouts
   - TypeScript strict. NEVER `any`. IDs and timestamps = `string`
   - Permissions: `can("mod.action")`, `<Authorized permission="...">`
   - Run: `cd frontend && npm run test -- --run {test-file}`
   - **Verify: PASS**
   - RULE: Every component in `modules/{mod}/components/` gets a `.stories.tsx`
   - NEVER: Component without at least a Default story at commit time
   - RULE: New SHARED components (`components/` or `components/ui/`) MUST have `.stories.tsx`
   - RULE: If Storybook not configured, configure BEFORE creating shared components. Check: `ls .storybook/main.ts`
4. **REFACTOR — Clean if needed.** Run full suite: `npm run test` — ALL PASS
5. **GUARD:** `npm run lint && npm run typecheck`
6. **COMMIT:** `cd frontend && git add [files] && git commit -m "feat(mod): description"`
6b. **POST-TASK VALIDATION:**
   - [ ] Colors/spacing/typography from DESIGN_SYSTEM.md tokens only (no hardcoded hex/px)
   - [ ] Component structure per ARCHITECTURE.md module pattern
   - [ ] Imports use `@/` alias, no barrel exports
   - [ ] Labels from `lib/labels.ts`, not hardcoded strings
   - [ ] Shared components in `components/`, module-specific in `modules/{mod}/components/`
   - [ ] New shared component → has `.stories.tsx`
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
