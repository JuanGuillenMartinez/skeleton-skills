---
name: skeleton-implementing
description: >
  Use when a plan exists and is ready to execute. Routes each task to the
  appropriate stack-specific implementing skill (Laravel or Next.js) and
  runs architecture guards after each task completion.
---

# Skeleton Implementing

Router that executes plan tasks in order, delegating each to the appropriate stack-specific skill.

**Announce:** "Using skeleton:implementing to execute the plan."

**Prerequisite:** Plan exists in `docs/plans/` and has been approved.

## Checklist

1. **Locate and read the plan** in `docs/plans/`
2. **Verify prerequisites:**
   - Dependencies installed (composer packages, npm packages)
   - Docker/Sail running (if backend tasks exist)
   - Dev server accessible (if frontend tasks exist)
3. **For each task in the plan, in order:**
   a. Read the complete task definition
   b. Determine target repo:
      - Path starts with `Modules/` or `backend/` → backend
      - Path starts with `modules/`, `src/`, `app/`, `frontend/` → frontend
   c. Activate stack-specific skill:
      - Backend → `skeleton:laravel-implementing`
      - Frontend → `skeleton:nextjs-implementing`
   d. After completing the task: activate `skeleton:architecture-guarding` for the repo
   e. Mark task as completed in the plan (checkbox `[x]`)
4. **After ALL tasks complete:** Activate `skeleton:verifying`

## Correct Example

```
Plan has 5 tasks:
  Task 1: Backend migration — routes to skeleton:laravel-implementing
  Task 2: Backend Data + Actions — routes to skeleton:laravel-implementing
  Task 3: Backend endpoint — routes to skeleton:laravel-implementing
  Task 4: Frontend types + API — routes to skeleton:nextjs-implementing
  Task 5: Frontend hook + page — routes to skeleton:nextjs-implementing

After each task: skeleton:architecture-guarding runs
After task 5: skeleton:verifying runs
```

## Incorrect Example

```
Plan has 5 tasks:
  Agent processes all 5 tasks in a single batch
  ❌ No architecture guard between tasks
  ❌ Backend and frontend tasks mixed without order
  ❌ Checkboxes not updated in plan
```

## Files Read

- `docs/plans/*.md` — the active plan

## Files Generated

None directly. Delegates to sub-skills.

## Transitions

| Condition | Destination |
|-----------|-------------|
| Task is backend | `skeleton:laravel-implementing` |
| Task is frontend | `skeleton:nextjs-implementing` |
| Task completed | `skeleton:architecture-guarding` → next task |
| All tasks completed | `skeleton:verifying` |
| Task blocked | Escalate to human |
| Guards fail 3x on same task | Escalate to human |
