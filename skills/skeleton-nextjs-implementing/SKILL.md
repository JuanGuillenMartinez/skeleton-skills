---
name: skeleton-nextjs-implementing
description: >
  Use when implementing a frontend task from a plan. Enforces TDD with
  Vitest, Testing Library, and MSW in the modules/ architecture. Uses
  TanStack Query patterns, never vi.mock or direct fetch().
---

# Skeleton Next.js Implementing

TDD implementation for frontend tasks. RED→GREEN→REFACTOR→GUARD→COMMIT for every task.

**Announce:** "Using skeleton:nextjs-implementing for this frontend task."

**Prerequisite:** Task definition from plan (files, code, expected output).

## Checklist

1. **Read the current task** from the plan
2. **RED — Write failing test:**
   - Location:
     - Hooks/API: `modules/{mod}/__tests__/{file}.test.ts`
     - Components with logic: `modules/{mod}/__tests__/{component}.test.tsx`
     - Lib/utils: `lib/__tests__/{file}.test.ts`
   - Framework: Vitest + Testing Library + MSW
   - Use MSW to intercept API calls (NEVER `vi.mock`)
   - Run: `cd frontend && npm run test -- --run {test-file}`
   - **Verify: FAIL**
3. **GREEN — Minimal implementation:**
   - Respect CLAUDE.md conventions:
     - Files: kebab-case
     - Data fetching: `api.*` from http-client (never direct fetch)
     - Query key factories: `{mod}Keys.all`, `{mod}Keys.list(params)`, `{mod}Keys.detail(id)`
     - Hooks: `use{Mod}s`, `use{Action}{Mod}`
     - Labels: from `lib/labels.ts` (never hardcoded strings)
     - TypeScript strict, never `any`
     - Components: hooks = behavior, components = UI
   - Run: `cd frontend && npm run test -- --run {test-file}`
   - **Verify: PASS**
4. **REFACTOR — Clean if needed:**
   - Run: `cd frontend && npm run test` (full suite)
   - **Verify: ALL PASS**
5. **Lint + Type check:**
   - `cd frontend && npm run lint`
   - `cd frontend && npm run typecheck`
6. **Commit:**
   ```bash
   cd frontend && git add [files] && git commit -m "feat(mod): description"
   ```

## Test Patterns

### Hook test with TanStack Query + MSW

```typescript
import { renderHook, waitFor } from '@testing-library/react';
import { http, HttpResponse } from 'msw';
import { server } from '@/test/setup';
import { useCustomers } from '../hooks/use-customers';
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

### Permission test

```typescript
it('hides create button when user lacks permission', () => {
  render(<CustomerList />, {
    wrapper: createWrapper({ permissions: ['customers.view'] }),
  });

  expect(screen.queryByText(labels.common.create)).not.toBeInTheDocument();
});
```

### Form submission test with error handling

```typescript
it('displays validation errors from API', async () => {
  server.use(
    http.post('*/api/customers', () =>
      HttpResponse.json(
        { error: { code: 'validation', message: 'Validation failed', details: { name: ['Required'] } } },
        { status: 422 }
      )
    )
  );

  render(<CustomerForm />, { wrapper: createWrapper() });

  await userEvent.click(screen.getByRole('button', { name: labels.common.save }));

  await waitFor(() => {
    expect(screen.getByText('Required')).toBeInTheDocument();
  });
});
```

## Red Flags — Anti-Rationalizations

| Agent's excuse | Reality |
|----------------|---------|
| "It's just a pure UI component" | If shadcn/ui with no custom logic: skip test. If it has hooks, data, or permissions: TEST. The line is clear. |
| "MSW is too much setup for one test" | MSW is configured once in `test/setup.ts`. Each test only adds specific handlers. The setup already exists. |
| "I don't know how to test hooks with TanStack Query" | Wrapper with QueryClientProvider + `renderHook`. Pattern documented in ARCHITECTURE.md. |
| "TypeScript already validates data shape" | Types validate structure. Tests validate behavior. They're complementary, not substitutes. |
| "I'll write the test after" | No. Delete the code. Start over. Test first. Always. |
| "It's just layout/routing, it can't be tested" | Layouts with AuthGuard or permissions ARE tested. Layout without logic: skip. |
| "Component tests are fragile" | Fragile tests test implementation (IDs, CSS classes). Robust tests test behavior (visible text, ARIA roles). Test behavior. |
| "api.ts only calls http-client" | Correct, but the hook that consumes api.ts has logic (invalidation, error handling). Test the hook. |
| "This is a one-off page, testing is overkill" | One-off pages with bugs are one-off bugs. Tests prevent them. Write the test. |
| "I need to see the UI first to know what to test" | You know what behavior to expect from the spec. Test the behavior, not the pixels. |

## Correct Example

```
Task: Create useCustomers hook

1. ✅ Write test: it('fetches customers list', ...) with MSW
2. ✅ Run test → FAIL: "useCustomers is not defined"
3. ✅ Create: modules/customers/hooks/use-customers.ts + modules/customers/api.ts
4. ✅ Run test → PASS
5. ✅ Run full suite → ALL PASS
6. ✅ lint → clean, typecheck → clean
7. ✅ Commit: "feat(customers): add useCustomers hook with query key factory"
```

## Incorrect Example

```
Task: Create useCustomers hook

1. ❌ Create hook first (no test)
2. ❌ Use direct fetch() instead of api.get
3. ❌ Hardcode query key as ["customers"] without factory
4. ❌ Use vi.mock to mock http-client
5. ❌ Skip typecheck
```

## Transitions

| Condition | Destination |
|-----------|-------------|
| Task completed (commit done) | `skeleton:architecture-guarding` (frontend mode) |
| RED phase test passes (shouldn't) | Rewrite the test |
| GREEN requires >100 lines | Task too large, decompose it |
| Full suite fails | Investigate before continuing |
| Task blocked | Escalate to human |
